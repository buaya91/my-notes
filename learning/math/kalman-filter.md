# Motivation

Informally, kalman filter helps us to improve estimation of data when
measurement is subject to noise.

3 basic concepts to understand

### Measurement (Input data)

It usually take form of `(t -> data)`, where `t` indicates time, and `data`
can be a vector.

We typically want to filter time-series data, but it can also be
repeated measurement, the concept is to combine multiple reading to figure
out the error pattern, and thus able to get better estimate of value.

Data collected, it is inaccurate due to noise and also lack of
accuracy due to collection mechanism, ie. inaccurate sensor

typically denoted as `Z`

### Prediction (Input, transformation)
Our prediction of next measurement, ie. T+1, it is based on
the measurements and estimations we've made so far, ie. `0 - T`

`Question: How to determine the initial formula for it? ie. matrix`

### Estimation (Output)
Our estimation of actual value, based on measure, generally it will be
different from measurement, as we assert that measure is not accurate

Est N = (KalmanGain * Measurement) + (1 - KalmanGain) * Est N - 1


N is state ?? (Can be think as time step, k=1,2,3)


1. What is KalmanGain?
    - Kalman Gain is what Kalman filter provide?
2. How to specify Estimation
    - A matrix where size depends on your state
3. How to specify Measurement
    - ???


## Main reference
https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python
http://bilgin.esme.org/BitsAndBytes/KalmanFilterforDummies

----

## More notes

Some example assume you know the error, eg, you know the drift of a sensor,
then you can deduce. A basic premise is that

> An inaccurate measurement can increase accuracy of result

why?? Because more information is always more informative (surprising eh?)


We always make prediction **before** estimation, as estimation is an
correction which depends on prediction and previous estimation, while
prediction typically only depends on previous estimation.


-----

Idea: use train system to explain what Kalman Filter is and why is it useful

