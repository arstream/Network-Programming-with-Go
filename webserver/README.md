# Chapter 10 A Complete Web Server 

This chapter is principally a lengthy illustration of the HTTP chapter, building a complete Web server in Go. It also shows how to use templates in order to use expressions in text files to insert variable values and to generate repeated sections. 

## Introduction

 I am learning Chinese. Rather, after many years of trying I am still *attempting* to learn Chinese. Of course, rather than buckling down and getting on with it, I have tried all sorts of technical aids. I tried DVDs, videos, flashcards and so on. Eventually I realised that there *wasn't a good computer program for Chinese flashcards*, and so in the interests of learning, I needed to build one.

I had found a program in Python to do some of the task. But sad to say it wasn't well written and after a few attempts at turning it upside down and inside out I came to the conclusion that it was better to start from scratch. Of course, a Web solution would be far better than a standalone one, because then all the other people in my Chinese class could share it, as well as any other learners out there. And of course, the server would be written in Go.

The flashcards server is running at [cict.bhtafe.edu.au:8000](cict.bhtafe.edu.au:8000). The front page consists of a list of flashcard sets currently available, how you want a set displayed (random card order, Chinese, English or random), whether to display a set, add to it, etc. I've spent too much time building it - somehow my Chinese hasn't progressed much while I was doing it... It probably won't be too exciting as a program if you don't want to learn Chinese, but let's get into the structure. 