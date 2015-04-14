---
layout: post
title: Interviewing with Mozilla for a summer internship
---

How I got an offer for an internship with Mozilla

-----

Hurrah! A lot of things happened since my last post. I haven't had much time to do some progress on an awesome article that I've been preparing for quite some time now, but trust me, it's going to be out by December 28th, and it's going to be awesome! It's my little Christmas gift for my readers.

Today, I am here to write about my pleasant experience with Mozilla's interviewing and recruiting process.

Now, let's start this out:

Hey, you there, computer science student! Look at me! It's recruiting season. Companies are looking for talented candidates for summer internships. In fact, recruiting began around September, so I advice anyone who is currently looking for a summer internship to apply asap. Positions are typically filled in a first come first served basis, so pay attention, stay tuned, and apply immediately. Welcome to The Real World.

Until a while ago, I, too, was looking for a summer internship. Over the course of time, I have gotten especially picky about the companies I apply for, because I really want to work on something that I truly enjoy. Life is just too damn short not to love your job. Readers of my blog will surely know that I am passionate about low level programming, operating systems development, compilers, and any other hardcore, C-related, highly algorithmic and brain-intensive task. Isn't that beautiful?

Anyway, I decided to look up for opportunities matching my personal interests and I came across Mozilla's FirefoxOS, or B2G (Boot2Gecko) project. You can read about it in wikipedia. Basically, it is Mozilla's attempt to enter the mobile market with a mobile operating system based on open web standards, with no proprietary bullshit, where apps are developed in CSS, HTML5 and JavaScript. 

The code name Boot2Gecko, although it certainly sounds shiny and fluffy, is a little bit out of the scope: Gecko is just the layout engine, you know, a program that takes marked up content (HTML, XML, etc.) and formatting information (CSS, ...) and displays the formatted content on the screen - it is usually part of the browser. Obviously, you can't exactly boot to a layout engine, so that's why Gecko sits on top of Gonk, which is the underlying Linux-based kernel and hardware abstraction layer based on AOSP (Android Open Source Project, the Linux-kernel based OS that Google assembled to support Android). But I'll get into the details later on. 

Let's start out by looking at Mozilla's recruiting process. This was a very interesting experience.

## Applying
Head over to [Mozilla careers](http://careers.mozilla.org/en-US/) and have a look at the internship opportunities. As of this writing, there are 12 internship positions available, all of them covering different fields. There's even a research opening.

Mozilla uses jobvite to manage candidates applications. If you don't have an account, you will have to create one. Now, the problem of making yourself different from the rest of other applications shows up. They get lots of applications; you need to shine in the dense universe of candidates.

Don't take my entire word for it, I'm really not an expert on this matter, but since I read *The Google Résumé* (great book by the way) and got an offer from Mozilla, I think I might have a (rough) idea of how you can improve your chances of being called in for an interview. Some of the points below were mentioned by my recruiter when I asked her what made my resume shine, others are just general tips that I find useful and think that contributed to make me a good candidate:

* **Send a 1-page resume, no more, no less** - This is really important. I can't stress this enough. Especially for positions in the USA, the standard, accepted and recommended form is a 1 page resume. In Portugal and other EU countries, there's usually a strong desire to show off a lengthy, 10+ pages CV with *The History Of My Life*. Seriously. Don't do that. *This is 'merica, bro!* Be pragmatic. Imagine you are on the other side, getting thousands of applications. Which one is more appealing: going over 1000 1-page resumes from 1000 candidates with relevant information, or going over 10,000 lengthy and boring pages of scattered information about Joe's card-playing hobby, or Andrew's wonderful chess-playing skills, oh, and there's Tom, who's got a driving license for cars *and* motorbikes. Yep. REALLY INTERESTING. Seriously, why do people put this in their CV? This is *SO* Europe. Submitting a long CV is half way to rejection simply because no one is really willing to extract the relevant information for this position from a 10 pages document. Plus, the more pages you add to your so-called CV, the more difficult it becomes to keep it impressive. You can have a wonderful career, and be better than everyone else, but you need to condensate the best of your career - the best of the best - into one page. That's it. **ONE PAGE**. It's possible. And by the way, did you notice that I mentioned CV and resume? CV is not very common in the US, forget about that. They really want something small, simple, to the point. 1 page, *that's it*.
* **Tailor your resume to the specific position you're applying to** - Don't keep a single, global, universal resume. This is a poor choice, because it is very easy to spot "generic" resumes. It is absolutely necessary to write a different resume for each position you apply to, because a resume should list skills relevant for the specific position. Unless you really have nothing better to include, there's no point in listing, say, an HTML certification if you're applying to a compilers development position. That's completely off the record. You have to narrow down your resume to a set of specific skills that match the job and show that you have the necessary knowledge. Also, submitting a generic resume sounds like you're blidnly mass-applying to every internship position you find, conveying the idea that you really don't care about the company or the project, you just want to find a 9 to 5 job to pay your bills.
* **Do not include useless and meaningless information like "Good communication skills"** - That's just stupid. Space in your resume is precious, don't waste it with meaningless entries like "Good communication skills", "Hard worker", or anything resembling to that. Anyone can write that, and it adds little to no information. This also includes things like "Good knowledge of Microsoft Word / Office", "Proficient in working with Windows / Linux", etc. These are very broad topics, and it is utterly irrelevant for a software development position. Do not include high school - that's irrelevant as well. Everyone did highschool.
* **List past projects related to the position you're applying to** - That's a good method. List related projects, even if you haven't touched them for quite some time. I listed some projects directly related to this internship that I haven't touched for 2 years. And so what? Well, I worked on it, if I did it before, I could do it again. Just make sure to give it a look before the interviews - they will ask you to describe it. But if you really did what you say you did, and if you enjoyed it, then you won't have any problem discussing it even after 2 years.
* **Show passion** - Do you own a blog? Put it in your resume! Yes, I *proudly* included codinghighway in my resume, and the recruiter visited it, went through some of my posts and liked it. Even though most of them are highly technical, they give a strong hint about my personality. For example, if you read the *About Me* section, you will notice that it perfectly fits Mozilla's philosophy. Believe it or not, I didn't do it on purpose - I haven't touched that page for months - but it really tells a little bit more about me and the idea behind this website.
* **Be yourself** - Just be yourself. Ok, I must confess, I did something that goes against *The Google Resume* recommendations: I wrote my resume using the first person. Why? Because I don't like the idea of having to struggle all the time to find alternate ways of exposing the information without talking of myself as "I". I don't like excessively formal stuff. Why complicate? I just ended up doing it my way, and ignored the usual advice. Add a personal touch to your resume, make it *you*. After all, you're selling yourself.

And then comes the cover letter. Always include a cover letter, it never hurts. Use it to talk a little bit about yourself, why you think this position is a good fit for you, why you would be a good addition to the company, what makes you want to work there, etc. Make sure to perform some research on the company before, like reading about their products and their philosophy. Obviously, this implies that the cover letter be tailored to this specific role as well.

Lots of other useful tips and tricks are available on *The Google Résumé*, by Gayle Laakman McDowell. Give it a look, it's totally worth it.

After applying, I didn't hear back from Mozilla for about 3 weeks. I don't remember exactly, but eventually, I got back an e-mail to set up a Skype interview. 

Every candidate is assigned a recruiter. The recruiter is there to go through the application, tell you about your performance in interviews, and answer any questions you might have about the whole process. From the candidate's point of view, the recruiter is the face of the company.

The recruiter that took care of my application was [Kimber Schlegelmilch](http://www.linkedin.com/profile/view?id=65426766&authType=name&authToken=ToG2&trk=prof-sb-browse_map-name).

## The interviews

Once they reach out to schedule your first interview, the whole process will be really quick. You are likely to have a final decision within 2 weeks after your first interview, they are usually pretty fast to give you feedback and schedule interviews quickly. For example, if you have an interview on Monday, you will know the result by (at most) Tuesday, and they will want to schedule the next interview for Wednesday or Thursday.

And then, of course, comes the fun part: solving the technical challenges. I've had 3 interviews with different Firefox OS engineers. Depending on how your first interview goes, they can schedule between 2 or 3 more interviews.

Now, for the sake of it, I am not going to post here the specific questions. Let's put it like this: most of them were related with the kind of stuff you can find on this blog: recursion, pointers, linked lists, bit manipulation, algorithms, you get the point. Anyone with a solid computer science background and good coding skills can easily make it through. And let me tell you something: it's not *that* hard. You do not have to come up with highly complex algorithms. You won't be asked to implement things like [push-relabel](http://en.wikipedia.org/wiki/Push%E2%80%93relabel_maximum_flow_algorithm, or [maximum bipartite matching](http://en.wikipedia.org/wiki/Matching_%28graph_theory%29#Maximum_matchings_in_bipartite_graphs). The problems are typically reasonable; the goal is to see how you tackle them rather than seeing how many algorithms you know. Do you immediately rush to code, or do you try to get a global vision of the problem in the first place? Do you ask questions to make the problem clearer? Can you split the problem into smaller problems and deal with one at a time? Are you able to find bugs in your solution? Can you fix them? Are you aware of your solution's limitations? Do you make any assumptions? If so, did you document that (a simple comment will do it)? Can you make it more efficient?

Remember, it's all about how you think and how you deal with problems rather than how your code looks like. You don't *have* to code a solution in 1 second, and in fact, doing so might actually hurt you. Interviewers know how hard a problem is supposed to be, if you immediately spit the answer, they will assume you knew the question and you are trying to cheat. Better be honest and say you've heard it before - that's huge! Huge, huge bonus point. Honesty is very important.

In general, the technical questions were around linked lists, bit manipulation, multithreading, and arrays.

## 1st interview
My first interview was with [Mike Habicher](http://www.linkedin.com/pub/mike-habicher/49/437/b7b), who will be my mentor. You can see some pretty impressive entries on his LinkedIn profile, like "Designed and implemented sockets API" and "Designed, implemented, and optimized embedded IP stack". That's cool, but that's also a little bit scary from the interviewee's standpoint.

We've had a very interesting chat that lasted for about 1 hour and 20 minutes. The first 40 minutes or so were entirely dedicated to my resume, which I wasn't expecting, but nevertheless it was quite a good experience. He seemed to be particularly interested in my previous compiler's project. I haven't touched that project for 2 years (it was part of a 2nd year compiler's course), but luckily, I studied it before the interview. You will be asked tons of questions about the entries listed in your resume, so make sure you really know what you're talking about, and have a look at older projects. Don't list what you don't want to be asked about. Oh, and by the way, you don't have to review your old projects until the very last detail, 1 or 2 hours should be enough to get on track: review the basic system's architecture, recall the hardest bugs to find and fix, think about unusual code you had to write, and things like that.

For the second part of the interview, I was given 3 technical questions with increasing level of difficulty. All of the questions were about linked lists and bit manipulation.

## 2nd interview
Michael Wu was the second interviewer. This time, we didn't really talk very much about my resume, he jumped right into technical questions. He asked me if I knew about UTF-8, UTF-16, Unicode, and other charsets. I explained the general concept and idea behind Unicode and UTF-8, but since I never really had to deal with that, he didn't go any further. The technical questions had to do with bit manipulation and multithreading. The multithreading question was cool and challenging (nice one, Michael!).

## 3rd interview
This time, I had the pleasure of talking to [Hema Koka](http://www.linkedin.com/pub/hema-koka/1/a95/83b), the team lead engineer. Although it was also a technical interview, this time it was centered in my resume. At Mozilla, they really want to make sure you know what you're talking about, and that you really worked hard on the projects in your resume. You may be asked what was the most complex project you've worked on, how many elements were on the team, if you have ever done mobile development, if you have unit tests in your projects, describe object oriented concepts like inheritance, explain design patterns you used in project `X`, what classes are you taking this semester and what are they about, what project are you working on at the moment, why Mozilla, why open source, what do you like (embedded stuff? low level programming? web development?), and so forth, you get the point.

## The offer
I got offered a position to spend my summer with Mozilla as an intern in the FirefoxOS media apps team, in their San Francisco office. The recruiter called me on Skype to discuss the details of the offer. I get free housing within a walking distance of the office, and Mozilla covers every relocation cost, including the flights. Every intern is given a laptop (they're getting me a Macbook Pro 15-inch Retina, 16 GB RAM, i7 2.6 GHz Quad Core with a 512 GB SSD drive - whaaaaat?! Yeah, that's right!), and it's a very competitive pay: 6.2K/month. Pretty good for an intern who doesn't have to pay rent and has all relocation expenses covered. I start on June 30th, and the internship lasts for 12 weeks (30/06/2014 - 19/09/2014).

By the end of the internship, I will be making a presentation about the work I developed, which will be broadcast over air mozilla. There are weekly team meetings, also broadcast on air mozilla, as well as corporate meetings. Once every week, the teams get together for catered lunch.

I will mostly be working with C++, with Mike Habicher mentoring me. The team is currently working on the Camera API (you see, the libraries that apps developers will use to control the phone's camera), but until late June, priorities can change.

For a C expert, C++ is not very difficult to get around with; although there are some core differences (the languages evolved independently over time), the underlying basics of C are always there. I ordered my copy of The C++ Programming Language, since I intend to master the language before touching the code base. If you want to make an amazing job, you have to master your tools, right? This is true for every job. Mike recommended me to learn the basics of Javascript as well. This is because of the way that FirefoxOS is structured. On the core of FirefoxOS sits Gonk, which, roughly speaking, is a tiny Linux kernel based on AOSP - Android Open Source Project, the kernel packed by Google that runs on Android devices. It uses a custom version of libc called *bionic*. On top of Gonk comes Gecko, the free and open source layout engine used in many applications developed by Mozilla Foundation. And then, wrapped around Gecko, there's Gaia, the apps layer, written in HTML/CSS/JS. This is what most of FirefoxOS apps developers see and deal with.

Obviously, HTML, CSS and JS for themselves cannot give you access to lower level routines and system functions that allow you to, say, manipulate the phone's camera, or access GPS. Thus, the FirefoxOS engineers work on a set of C++ library packages that provide access to these low level actions that useful apps will typically want to execute. The connection between the C++ APIs and the apps on HTML is done through Javascript. This means that automated tests tend to be written in Javascript, and that's why I should learn some of it.

## A summary of my interviewing experience
After being called in for the first interview, the whole process was pretty fast. My recruiter did an amazing job and was always there to answer my questions. More than once, I talked to her on IRC instead of sending an email and waiting for the reply. There are very few companies where this is possible - where else can you chat live with your recruiter any time you want on IRC? That's nice!

I can say that my experience with Mozilla was very positive. They work fast, and developers / interviewers are good professionals.

I am thrilled to have an offer from Mozilla and to participate on a project with such a group of smart people, and to contribute to a more open web.

Always prepare yourself for interviews. Technical questions are important, but make sure you can talk about your resume and projects you listed without hesitation. You will also be asked about the current project you're working on.

If you are a San Francisco intern, feel free to ping me! Check the About Me page, or find me in Mozilla's IRC under the nickname Fill.

See you around!
