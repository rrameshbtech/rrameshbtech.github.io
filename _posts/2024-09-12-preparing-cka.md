---
layout: post
title:  "Preparing for CKA"
date:   2024-09-13 00:00:00 +0530
categories: experience exams
tags: cka exam assessment kubernetes preparation
published: true
excerpt_separator: <!--more-->
---
![Certified Kubernetes Administrator](/assets/images/cka-logo.png "cka logo in blue color")

Usually, my conversations with friends start at one end of the world and end at the otherside. It was nothing different at that time. We were seriously discussing certifications and blaming the industry for using it as a checkbox. The discussion took a U-turn, and we started debating how useful it could be. This led to checking what are the vauable certifications which are really valued by the industry and we zeroed in on Linux foundation certifications. We decided should try one their certifications and we chose `Certified Kubernetes Administrator`. I volunteed to be the guinea pig then I immediately purchased the exam.
<!--more-->
## Hibernation and a wakeup call
It's been 5 to 6 months. One day, I noticed an email from unfamiler sender. Damn! It's the Linux Foundation. I totally forgot about the certification I purchased, and I was lost in my routines. Yes. I still have more than 6 months to finish it. But I thought this could be my last wakeup call. If I skip this time, either it might struggle at the last moment or miss it. Without the second thought, I booked the exam slot immediately.

Thanks [Brian Tracy](https://www.briantracy.com/) for [Eat That Frog](https://www.amazon.in/Eat-That-Frog-Great-Procrastinating/dp/152309513X/ref=sr_1_3?crid=9B2WAG6OT1YA&dib=eyJ2IjoiMSJ9.C5mvU-hYM_3aawLyI9CSmt908HQAK7KEPyVzjr4QqZ6rreskBTK3U-7ENpQiI9iuEtAWu2RUSsnDCYNtf5ePwWJ7AnCOmJfK_aqqSpMwFZ8kQfe92NiKpAqoMbXFhcdwxICfygu3gfcR4hjGOsp8tAnmRTU1hILSe9zxqV-Sguy14cdQfb5IwEggn1jiP4p8ObbxQkmkDHj53w2F5bT1ZbmQrkzDJjBXwWtdY1xTaUk.L0KVI4MIXxeIXpQiJ40JXerLF9sCEyDY5Y2XBeNLDXU&dib_tag=se&keywords=eat+that+frog&qid=1725962887&sprefix=eath+that%2Caps%2C260&sr=8-3)

## Move like a snail
Though I work with Docker and Kubernates often, Kubernetes is not my bread and butter. It is just another tool to make our lives easier. So I never learned to focus on them. I took this as a chance to build a Kubernetes knowledge base for myself. Easier said than done. It wasn't that easy to squeeze in something new in already packed schedules.

So the first thing I did was to create an Ulyssis pact. I informed my family & friends about the exam. Ta-da! It did the magic. Now I got watchers & trackers for my goal. Especially the kids, they enjoy monitoring their parents.

Next, I rearranged my routines to create a schedule for my exam preparations; this varies from dropping kids off at school to morning walks and more. But this made me think about what I could delegate and what I should own. I should thank my wife and kids who helped me make these shuffling. Once they knew my goal, they were eager to help me and it was smoother than planned.

Now comes the important part: I need to start my preparation. I never believe in preparing just for exams. So I decided to start with the basics; I signed up for the madatory courses first. The 2 important courses that helped me are
1. [Introduction to Cloud Infrastructure Technologies](https://courses.edx.org/courses/course-v1:LinuxFoundationX+LFS151.x+2T2018/757af1dd1fee425aadf327a4f8502017/) By Edx 
2. [KodeKloud's CKA course](https://learn.kodekloud.com/courses/udemy-labs-certified-kubernetes-administrator-with-practice-tests)

It was a slow start; sometimes I felt I was going slower than a snail, but I was happy that I was moving. One thing I was stubborn was to study at least one topic per day. It really worked. Sometimes a 10-minute start led to 1 hour session.

## Preparing for the battle
Though I moved like a snail, I was in a better position as the exam approached. But it wasn't enough. Now I had to increase my study time. My family helped me again just like they would help the kid preparing for a public exam :D.

I completed the courses planned and practiced enough to get things done. But I know the exam isn't just that. They have controlled environments and specific mechanisms to measure. Maybe the CKA exam is less different. But I still had to understand what to expect on the table. Luckily, there are a lot of videos and articles about dos and don'ts. They can be as intimidating as they are helpful, so be careful when you are going through them. Their caution about speed and using tools (like tmux, jq and vim) caught my attention, and I had never used them before. Now I can't step back from the exam (remember the Ulysses Pact), and during the discussion, my friend encouraged me saying I would be okay. Now again, I started preparing myself with those tools as much as I could.

During the last few days, I spent time practicing and running through the notes (I followed the Zettlekaston method; trust me, it is a gem of a technique to build a knowledge base). Another most important thing that helped me was Killer.Sh's mock exam and environment. You get this free along with the exam purchase. They help you practice for exam in identical environment. Usually, we get two 36-hour sessions and one mock exam set. I only used one session and never used the second. But I will recommend planning early to use both sessions. It will be very helpful in your preparation.

The next important thing is preparing yourself and your environment. As it is an online performance-based exam, the Linux Foundation takes a lot of precautionary measures to keep the integrity of the certification. I recommend going through their [instructions](https://docs.linuxfoundation.org/tc-docs/certification/tips-cka-and-ckad) very carefully and ensuring that you have the proper setup for the exam. This helps you avoid unnecessary drama during the exam.

I had a few difficulties writing the exam from my room. First, as per their rules, there shouldn't be any papers or writing materials in the surroundings when writing the exam. But my room is filled with books and has a lot of posters and charts on my walls. Second, just a few days before, my router and UPS started acting wired. They were restarting unexpectedly, so I needed plan B. I chose to write the exam from my friend's home. Thanks a lot to him and his family for their support. So I recommend preparing yourself for the exam environment as early as possible.

## The judgement day
As I mentioned, I already tested that my laptop supports PSI-secure browsers and other needs as given in the instructions. They provide an option to run automated tests on your laptop, verifying whether you have all the necessary access to support the exam environment.

The exam link opens 30 minutes before the scheduled time. I recommend starting right away, as the check-in process takes nearly 30 minutes, and you may also need to wait for the proctor to become available. My check-in process was smoother and did not have any issues.

But the exam experience wasn't that smooth. I explained these issues clearly in the sections below. After my exam was over, I wasn't very confident, knowing what I could have done better and what I actually did. But I had to wait for a day to get the result. As mentioned earlier, it is a performance-based exam. Though most of the evaluation process is automated, there are many ways to solve the problems, so manual evaluation is required, which takes up to 24 hours. I was prepared for any result and set my mind for the reappearance also. I was confident that if I had to reappear, I could easily clear it.

Hurray! I passed the exam. What a ride it was! Though it was a relief, I'm going to miss the thrill of the chase. Now I have found my next goal. This is not the hardest exam in the world. But this gave positive vibes and helped build my confidence in Kubernetes.
Thanks to my family, my friend, and his family for their help on this journey.

## Learnings & sharings
I collated all the important notes, which could be helpful for anybody who will write the CKA exam. Please comment below if you have further questions.

### Getting started
- Be cautious when you choose the certification. You may go through lows and highs during preparation, as our official or personal situation might change quickly. During those times, our reasoning helps us regain confidence and build conviction.
- Build your Ulysses pact. Usually, we do a lot of things in parallel, which diverges us from our goal. To avoid it, we can share our goals with family and friends. They remind us to stick to our goal.
- Start early and start slow. As the Linux Foundation gives us one year to write the exam, it is very common to put it in our backlog and forget. In my case, the reminder from the Linux Foundation was a wake-up call. Starting preparation early helped me to avoid being nervous. 

### Preparation
- Understand the fundamentals. As always, there are easy ways to crack the exam. But it won't do any good. I recommend starting with identifying gaps in your knowledge and filling them.
- There are a lot of courses for CKA, but I found the below 2 courses interesting and helpful.
  - [KodeKloud course for CKA](https://learn.kodekloud.com/courses/udemy-labs-certified-kubernetes-administrator-with-practice-tests). Mumshad Mannambeth, the instructor of this course, is excellent at explaining the concepts. I really liked the networking section of the course. Even though the course is planned for the CKA exam, they cover the required basic concepts to ensure you get the crux of all the concepts. Thanks, Udemy, KodeKloud & Thooughtworks (for sponsoring Udemy)
  - [Introduction to Cloud Infrastructure Technologies](https://courses.edx.org/courses/course-v1:LinuxFoundationX+LFS151.x+2T2018/757af1dd1fee425aadf327a4f8502017/). In case you are newer to the cloud or want to refresh your cloud knowledge. This is one of the best, and it is free too.
- Practice, practice & practice. There is no alternative to hands-on experience. Both of the above courses encourage practicing. Utilize the practice labs provided in the CKA course very effectively. Even though CKA is an open-book exam, it is time-bound. So practicing helps you solve problems quicker and better.

### Some tips & tricks
  - Read the questions carefully. If required, read it twice. Most of us read sentence by sentence, which results in misunderstanding the question or overcomplicating. So reading slowly and word by word helps you get a better understanding.
  - CKA is an open-book exam. You are allowed to refer to Kubernetes documentation. Though Kubernetes documents have all the answers you need, you will waste a lot of time searching unless you know what and where to look for.
  - Use imperative commands wherever possible. In 2 hours, you have to solve 17 problems. Commands help you create or update required resources a lot quicker. Another biggest advantage is their helpdocs. You can run the help command. This shows all the options and sample commands. I will recommend copying and pasting the sample command to ensure that you are not wasting time fixing typos.
  - Always copy and paste the given names. If we type ourselves, it is easy to make a typo. Do not give a chance for that.
  - Always set your context and namespace before starting to read the question. If no namespace is mentioned, it should be considered as "default" namespace. Missing the correct context is one of the common mistakes made by the candidates, including me. So set it even before you start reading the question. This becomes muscle memory and avoids unexpected errors.
  - If you are not comfortable, be brave to skip the question for later. As each question will vary in complexity, it is wiser to answer only the simple and well-known questions first. This builds the confidence and provides more room for the tricky questions.
  - Keep track of time. Sometimes, we think that a question will be easier. But when we start solving, it might get tricky and take more time. So it is good to keep track of time and skip to the next question if the current one takes longer duration.
  - If you are not already familiar with tmux, vim, and jq, it is good to have some hands-on. This will make your life easier during the exam. Vim is a must-have. The tmux and jq are good to have.

### Exam environment
Remember, practice grounds are similar to the real but not same.
  - Expect a slower working environment. PSI Secure Browser and Remote Desktop will be slower than our usual interfaces. Prepare yourself for the slowness. If not, our frustration will lead to anxiety and impact badly on results.
  - Practice the mock environment. Killer.sh's mock environment (which comes free with the exam purchase) has a near-real exam environment. Leverage this environment as much as possible.
  - Sometimes the remote desktop goes away with a reconnecting message. Be patient, as most of the time it comes back in seconds. Use the time to settle yourself.
  - The exam starts with the virtual tour. It gives you better clarity about the whereabouts of the exam environment. The clock starts ticking after you finish the tour. So make sure that you are comfortable before finishing the tour.


I hope this article would be helpful. Comment below if you have any specific question.
Thanks folks! Happy reading!

## Useful Links
- [CKA Exam page](https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/) provides a lot of deails about the exam.
- Courses
  - [Introduction to Cloud Infrastructure Technologies](https://courses.edx.org/courses/course-v1:LinuxFoundationX+LFS151.x+2T2018/757af1dd1fee425aadf327a4f8502017/) - A course for the beginners to get hands-on in Cloud.
  - [KodeKloud's CKA course](https://learn.kodekloud.com/courses/udemy-labs-certified-kubernetes-administrator-with-practice-tests) - Practice based course for CKA.
- Tools
  - [Vim](https://www.vim.org/) the basic text editor you will need to use during the exam. It is good to have hands-on in Vim.
  - [tmux](https://github.com/tmux/tmux/wiki) - Terminal multiplexer for splitting your terminal for working in parallel. This will be helpful when solving cluster upgrade, backup restore or troubleshooting questions.
  - [jq](https://github.com/jqlang/jq) - A handy tool to format and query the Kubernetes results. You can also prefer `-o=jsonpath` or `-o=custom-cloumns` command options.
- [Ulyssess pact](https://en.wikipedia.org/wiki/Ulysses_pact) is a freely made decision that is designed and intended to bind oneself in the future.
- [Eat that frog](https://www.amazon.in/Eat-That-Frog-Great-Procrastinating/dp/152309513X/ref=sr_1_3?crid=9B2WAG6OT1YA&dib=eyJ2IjoiMSJ9.C5mvU-hYM_3aawLyI9CSmt908HQAK7KEPyVzjr4QqZ6rreskBTK3U-7ENpQiI9iuEtAWu2RUSsnDCYNtf5ePwWJ7AnCOmJfK_aqqSpMwFZ8kQfe92NiKpAqoMbXFhcdwxICfygu3gfcR4hjGOsp8tAnmRTU1hILSe9zxqV-Sguy14cdQfb5IwEggn1jiP4p8ObbxQkmkDHj53w2F5bT1ZbmQrkzDJjBXwWtdY1xTaUk.L0KVI4MIXxeIXpQiJ40JXerLF9sCEyDY5Y2XBeNLDXU&dib_tag=se&keywords=eat+that+frog&qid=1725962887&sprefix=eath+that%2Caps%2C260&sr=8-3) by Brain Tracy
- [Zettelkasten](https://zettelkasten.de/overview/) a perfect way to build your knowledge base.