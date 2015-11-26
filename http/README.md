# Chapter 8 HTTP

## Introduction

The World Wide Web is a major distributed system, with millions of users. A site may become a Web host by running an HTTP server. While Web clients are typically users with a browser, there are many other "user agents" such as web spiders, web application clients and so on.

The Web is built on top of the HTTP (Hyper-Text Transport Protocol) which is layered on top of TCP. HTTP has been through three publicly available versions, but the latest - version 1.1 - is now the most commonly used.

In this chapter we give an overview of HTTP, followed by the Go APIs to manage HTTP connections.