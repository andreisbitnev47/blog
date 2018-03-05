---
layout: post
title: "0-Introduction"
subtitle: "What this blog is all about"
date: 2018-03-03 12:00
author: "Andrei Sbitnev"
header-img: "/img/mean_code.png"
cdn: 'header-off'
tags:
---
## Introduction

&nbsp; This blog is all about web development! The goal is to provide a step-by-step guide on building and deploying a <b>smart-house</b> project using JavaScript as the main programming language on both frontend and backend.<br>
Each post is a separate tutorial on some particular topic, and it starts where the last post left off. There will be a separate branch in a [github repository](https://github.com/andreisbitnev/smart-house) for each topic/post, so it is easy to continue at any step.
>Tip: for example for this post there is a branch [0-introduction](https://github.com/andreisbitnev/smart-house/tree/0-introduction), so you can clone it with `git clone https://github.com/andreisbitnev/smart-house.git`

## Software stack
<ul>
    <li>
        During the development as well as in production all the services will be <b>Dockerized</b>, so most of the examples presented in this course will be built using tools available on linux environment.
    </li>
    <li>
        The application to manage the devices by the end user will be built using the <b>MEAN</b> stack <b>(Mongo, Express, Angular 5, Node.js)</b>
    </li>
    <li>
        Wifi relay switches powered by the ESP8266 chip, will be programmed using <b>Mongoose OS</b>, and will also be executing JavaScript
    </li>
    <li>
        IOT devices will be communicating with the server through secure MQTT protocol. The <b>Mosquitto</b> MQTT broker will be user for this purpose
    </li>
</ul>

## Next up
Go ahead and check out the next post about building the project skeleton with angular cli and express generator [Angular 5 MEAN app](/2018/02/22/1-mean-app/)