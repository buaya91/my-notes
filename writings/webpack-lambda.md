## How to deploy javascript bundle to AWS Lambda?

AWS Lambda is a popular serverless service that allow developer to deploy stateless application without going through the hassle of managing infrastructure. For more, please read [AWS Serverless](https://aws.amazon.com/serverless/)

This article shows how to deploy bundled typescript/javascript to AWS lambda.

The official way to deploy an AWS Lambda is by using AWS Cloudformation `package` and `deploy` api, details are available [here](https://docs.aws.amazon.com/lambda/latest/dg/serverless-deploy-wt.html#serv-deploy)

The main downside of following the official guide is that it will package everything from your project root into a zip file and upload to S3. 
 
 The zip will include all files in your project root and more importantly the whole `node_modules` package which more often than not contains developer dependencies that is not required by the lambda.
 
The fundamental issue of this approach is that we are uploading more things than necessary which might be undesirable in some cases, eg:

* Project with many static assets like images for documenation
* Project that contains multiple entry point for different lambda, and wish to make use of dead-code-elimination before deployment
* Organization that publish AWS lambda very often, the code size difference can adds up quickly on S3

Javascript bundling is a common technique used by frontend developer to distribute code, we can leverage the same technique when deploying AWS Lambda by only deploy 1 single javascript that contains all dependencies it need and nothing more.

### Bundling via webpack

[Webpack](https://webpack.js.org/) is a popular software for javascript bundling, it also provides many additional feature like transpilation and tree-shaking (dead code elimination) which can be useful, for this article, I am going to focus on bundling. 

#### Install Webpack
Webpack is a CLI tool, so before we start, we need to install it 

```bash
cd <project_dir>
npm install --save-dev webpack
npm install --save-dev webpack-cli
```

the command above install webpack locally to your project, replace `--save-dev` to `--global` if you wish to install it globally

Webpack cli works by reading a config and perform bundling according to the config file, we are going to create config file next.

#### Create webpack config file

```ecmascript 6
const path = require('path');
module.exports = {
    target: "node",
    entry: {
        app: ["<your lambda entry point>"]
    },
    output: {
        // umd allows our code to be run by AWS Lambda
        libraryTarget: 'umd',
        path: path.resolve(__dirname, "<path_of_bundle>"),
        filename: "mylambda.js"
    },
    externals: {
        'aws-sdk': 'aws-sdk'
    }
};
```

This is a very simplistic webpack config file, the most important bit is the value of `module.exports.output.libraryTarget`, which has to be `umd` to be able to run on AWS Lambda environment


