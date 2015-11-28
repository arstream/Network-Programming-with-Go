# Chapter 14 Network channels


## Warning

The `netchan` package is being reworked. While it was in earlier versions of Go, it is not in Go 1. It is available in the `old/netchan` package if you still need it. This chapter describes this old version. Do not use it for new code.


## Introduction

There are many models for sharing information between communicating processes. One of the more elegant is Here's concept of *channels*. In this, there is no shared memory, so that none of the issues of accessing common memory arise. Instead, one process will send a message along a channel to another process. Channels may be synchronous, or asynchronous, buffered or unbuffered.

Go has channels as first order data types in the language. The canonical example of using channels is Erastophene's prime sieve: one goroutine generates integers from 2 upwards. These are pumped into a series of channels that act as sieves. Each filter is distinguished by a different prime, and it removes from its stream each number that is divisible by its prime. So the '2' goroutine filters out even numbers, while the '3' goroutine filters out multiples of 3. The first number that comes out of the current set of filters must be a new prime, and this is used to start a new filter with a new channel.

The efficacy of many thousands of goroutines communicating by many thousands of channels depends on how well the implementation of these primitives is done. Go is designed to optimise these, so this type of program is feasible.

Go also supports distributed channels using the `netchan` package. But network communications are thousands of times slower than channel communications on a single computer. Running a sieve on a network over TCP would be ludicrously slow. Nevertheless, it gives a programming option that may be useful in many situations.

Go's network channel model is somewhat similar in concept to the RPC model: a server creates channels and registers them with the network channel API. A client does a lookup for channels on a server. At this point both sides have a shared channel over which they can communicate. Note that communication is one-way: if you want to send information both ways, open two channels one for each direction. 