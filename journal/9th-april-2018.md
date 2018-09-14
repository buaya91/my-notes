### Time: 1310

* Discussed with Neil on what to pick up next, agreed on improving runbook
* Reported the possible cause of Lambda error, which is throttling due to some traffic driven API call
* Address Matt's comment on PUSER-2018 Unhealthy Host count alarm (Pending merge and test in dev)
  One way to test is to change the ASG type to EC2, then manually kill containers
  This might surface an error case of ZeroDivision