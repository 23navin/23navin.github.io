---
layout: page
title: Tekken 8 Rank Tracker
description: Python / OpenCV / PyTesseract / Flask / Celery / Redis
img: 
importance: 1
category: Personal
related_publications:
---
Recently, a Twitch streamer I watch has been playing Tekken 8 as a variety segment. Tekken is a 1v1 fighting game that is almost entirely skill-based and thus can be heavily analyzed. However, I couldn't find any websites or resources that track a players rank and their winrates versus each fighter. Coincidentally, I had been looking for a project centered around Python that would allow me to break out of my C/C++ and embedded software shaped shell. You may understand how I might try to combine my resolve to learn Python and my newfound passion of Tekken; the result is this project.

##### Project Goals

I wanted to be able to provide a youtube link of vod of someone (who will be referred to as 'player' throughout this page) playing Tekken into a python script, wait a little, then receive a summary of each match that was played in the video. For each match, this summary should include the following:

* Both the player and their opponent's fighter
* Both the player and their opponent's rank going into the match
* Whether the player won or lost the match
* How much the player's rating was adjusted after the match

My original goal was to

1. Develop a Python script that would produce the match data in csv form, and
2. Design a web interface using React where the data could displayed in an interactive way, suitable for analysis.

As this project is a Python learning exercise, I considered learning its syntax and unique quirks (affectionately called features by the language's community) as my ultimate goal.

##### Part 1 - The Algorithm

If you are inclined to pick through a stranger's code, here is the GitHub repo[(link)](https://github.com/23navin/Tekken8RankTracker/blob/main/src/T8RankTracker.py). Otherwise, I will do my best to explain it here in simple terms.

My algorithm is based on 'Finite State Machines' that I learned to build in SystemVerilog during my undergrad education.

##### Part 2 - Image and Text Recognition

This is the meat and potatoes of the project. Once I had my Python environment set up, the first thing I did was tinker with OpenCV's python library till I could reliably identify key elements within a Tekken vod.

##### Part 3 - Adding a web interface

sdfgs

##### How this project will rest



<div class="row">
    <div class="col-sm mt-3 mt-md-0">
            {% include figure.html path="assets/img/teamphoto.jpg" title="Team Photo" class="img-fluid border border-dark rounded z-depth-1" %}
        <div class="caption">
            Most of the CubeSTEP members with our adivising professor, Navid Nakhjiri, who is holding a 3d model of our satellite!
        </div>
    </div>
</div>

Mission Statement

1. Develop a generic, yet fully-featured, CubeSat platform that future CPP projects can use as a foundation to advance their research.
2. Utilize additive manu
