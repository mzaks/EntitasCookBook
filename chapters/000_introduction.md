# Introduction

### Hi 👋,

my name is Maxim and I am a software developer. I stumbled upon ECS (Entity Component System) and Entitas (an implementation of ECS) late 2012 while working at Wooga - a Berlin based casual game studio. Since than I helped improve and port Entitas to different languages and build a couple of games and Apps with it myself.

To be honest with you, writing this book was not my idea. Fine people from Entitas-CSharp community pushed me in this direction and promised to help out in the process. As this is the first chapter I am writing, hopes are still high that I will get some help 😀.

# Who is this book for?

This book will cover ECS concepts. How those concepts are implemented in Entitas. What are the tools we build to make developing with Entitas a better experience. And last but not least, the recipes for building games and apps with Entitas.

If you are an ECS novice and want to understand what it is? This book is for you.

If you know ECS, but want to rather understand how to build software with Entitas? This book is for you.

If you know Entitas and would like to browse through ideas on how it can be applied in diferent use cases? This book is for you.

If you are already experienced with Entitas and would like to build some tools for it to make your life even easier? This book is for you.

If you want to port Entitas to another language? This book is for you.

If you think Entitas is crap and you want to implement your own ECS framework? Even in this case, this book is for you!

# Book structure

This book will have 3 main sections:

## 1. Ingredients
This section will contain the ECS concepts and the implementation of them in Entitas-CSharp. As the book idea originated in the Entitas-CSharp community, the code sample will be in C#. But I might add some side notes on how the same concepts were implemented in other members of Entitas family.

## 2. Appliances
Appliances are the tools we implemented in order to work with the ingredients. I will cover already established tools, but will also spill some ideas for the things which could be cool, but are still either in early development or just mindfarts which I had or heard over the last 5+ years. Maybe some of those will inspire you to implement them or have another idea which is even cooler.

## 3. Recipes
This section will cover real world problems and how they can be solved with Entitas. Here you will learn how to use ingredients and appliances to make delicious meals. This section will probably have the highest amount of chapters and will be a continuous work in progress. We will also grade the recipe and define how good of an idea this recipe is.

# Why should I care about ECS?
Entity Component System is a simple pattern, which has a single core principal: _Separation of State and Behaviour_. This core principal is often a prelude to what people call _Data Oriented Design_. Which leads to the ideas of _mechanical empathy_, which you need to have, to perform hard core performance optimisations.

This is however not what attracted me to ECS five years ago. What I found fascinating is the ability to compose things. In ECS you are composing state and you are composing behaviour. Your application design becomes a set of lego pieces which you can stick together. If something is missing, you can easily create a specialised piece to close the gap. This approach helps you improvise. It enables you to implement software in an "Yes and..." manner. __Yes__ this thing is a car __and__ it can fly if we collect a special power up. __Yes__ this application stores all its data on disc __and__ it can also be synced with a server, showing the change in UI in real time without user needing to tap a button.

There is a saying: "Nothing is certain but death and taxes". I am so bold to say - __change__ is as certain as death and taxes. Specially when it comes to software and game development. We need to be able to improvise and try things out, in order to get to the best user experience. This is exactly why I found ECS so usefull. It helps me to build __production quality rapid prototypes__ and keep _prototyping_ while in production. I also find it fascinating, how _system thinking_ helps me model and solve difficult problems with highest degree of simplicity and integrate them on top of existing solutions.
