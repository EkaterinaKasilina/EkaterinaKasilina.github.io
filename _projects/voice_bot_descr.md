---
layout: project
title: 'Voice Agent for sales'
caption: Implementation of Voice Bot for call center
description: >
   This is how I implemented a Voice Bot as a POC which later become a successful project
date: 1 Sep 2020
image: 
  path: /assets/img/voice_bot.png
#links:
#  - title: Link
#    url: https://github.com/EkaterinaKasilina/written_digit_recognition?tab=readme-ov-file
#accent_color: '#4fb1ba'
#accent_image:
#  background: /assets/img/projects/digit_v4.png
#theme_color: '#193747'
sitemap: false
---

In 2019, before LLMs took the world by storm, my team and I were tasked with creating a 
voice bot for a telecom company’s call centers. 
The main problem was that when call center employees reached out to clients, the most common responses 
were clients saying they were not ready to talk, not interested, or simply no answers at all. 
Spending time on these calls was costly, so stakeholders decided to explore automation for handling them.

## Key Tasks 
My team was quite small, consisting of myself, another data scientist, and a Java backend developer. We had to
present convincing POC and if it would be successful we would get more money and bigger team. 
None of us had prior experience with this type of project, and we weren't entirely confident we would succeed. 
So, we began by identifying the key challenges we needed to address:

1. **Data collection**. We had no knowledge of what cold-call conversations typically sounded like. 
2. **Speach-to-text**. Converts spoken language into text.
3. **Natural Language Understanding**. Understand users answers.
4. **Dialogue Management**. Control the flow of the conversation.
5. **Text-to-Speech**. Convert text to speech
6. **Metrics**. How could we define if POC was successful?

## 1. Data collection
This was one of the most fascinating parts! 
At this stage, neither my team nor the stakeholders fully understood what aspects could actually be automated. 
We know nothing about call centers and stakeholders had no idea what ML could do!
We quickly realized that there were numerous types of calls, proposals, and scripts used by employees, 
so we began by listening to a sample of recorded conversations for one type of calls where clients were offered to switch to new contracts 
with better terms.   
After a few days of listening, we identified a segment that could be handled by machine learning. 
Conversations typically started with greetings, followed by asking if the client could spare 5-7 minutes. 
Many clients declined at this point, ending the call right there. But what if the client was willing to continue?
From here, it became clear that existing scripts were too complex for a machine learning model in the early POC stage.
After several brainstorming sessions, we created a simplified script:

1. We mapped a straightforward path for a successful call: if the client agreed with each statement and expressed interest in the proposal, the call would be transferred to a live agent for detailed discussion and confirmation.
2. If the client declined to talk, we would offer to call back later.
3. For any additional questions or unexpected responses, we would ask for clarification twice and then transfer to a live agent.
  
Time to implement! 

## 2. Speach-to-text
As I mentioned, our resources were limited, no way we could train complex models, so we began by exploring existing solutions.
After several trials, we found that only paid options provided the speed and quality needed for 
our speech-to-text task. Since this was a critical component, we had to allocate some budget to 
make it work effectively!(I intentionally don't mentioned what exactly was used as it was not for English language and hence is not relevant for many of you)


## 3. Natural Language Understanding (and again data collection)
This was another intriguing component! How could we quickly and accurately interpret responses? After another round of conversation analysis,
we observed that at nearly every stage, answers could be classified as "Yes," "No," or "Unclear." 
If a response wasn’t clearly a "Yes" or "No" — such as questions or hesitations — we classified it as "Unclear," 
prompting our voice bot to ask the client to repeat.  
This task essentially boiled down to building a simple classifier, even just using a set of regular 
expressions—but we had no data at all!  
To gather data, we created a chatbot that presented the phrases our 
bot would use at different conversation stages, showing them one by one without any specific logic. 
Then, we spent time responding to these prompts in as many ways as we could imagine. 
We also shared the chatbot link with colleagues and friends, asking them to spend 10-15 minutes answering the 
questions to expand our dataset.
The collected dataset was used to train our Naive Bayes models on top of TF-IDF text representation.

## 4. Dialogue Management
This was the most tedious part.
We designed a nested JSON structure to navigate between levels based on the client’s responses. 
It was both challenging and exhausting to account for every possible state, avoid looping, and accurately 
define all conversation levels and how each could transition to another.

## 5. Text-to-Speech
Last but not least! Our bot needed to use predefined responses — no stakeholder would allow it to say just anything! 
We had a set list of phrases to convert to speech. Using a Text-to-Speech model would have simplified development 
and future updates, but unfortunately, even paid solutions produced responses with odd, 
unnatural accents and misplaced emphasis. It sounded downright creepy! 
So, we recorded the phrases ourselves, adding natural pauses, different variations of the same phrases, sighs, friendly intonations, 
and even smiling as we spoke to make the bot sound more personable and human.

## 6. Metrics
We needed to determine how to demonstrate to stakeholders that our POC was effective, as our ML metrics
didn’t hold much business relevance. So, we created and secured approval for a list of metrics to track,
improve, and present to stakeholders:
1. The refusal rate should not be significantly higher than that of a real employee, accounting for the time of day and week.
2. The number of new agreements should meet the target level.
3. The average conversation duration should meet the agreed minimum.
Additionally, we developed internal metrics to monitor, such as the distribution of levels at which clients ended the conversation, 
the frequency of “Unclear” responses, and a few others.


## Run
Our initial run included just 20 calls. We carefully listened to each one, adjusted some phrases, and ran the process again. 
We repeated this cycle several times: making calls, recording, analyzing, refining phrases, saving data, and retraining 
the models, to continually improve the bot's performance.
After some number of iterations we successfully presented our POC and got a contract and budget for development!

# A few points we discovered through repeated analysis
The iterative process of running and analysing brought incredibly valuable! We uncovered insights that seem obvious in hindsight but helped to make improvments.
Here are just a few of them:
- Every voice bot phrase needed to end with a question, like "Do you agree?" or "Are you interested?" Without this prompt, many people simply remained silent.
- Proposal descriptions had to be concise; lengthy descriptions led to clients dropping off.
- We needed to detect if a client interrupted before the bot finished speaking, asking a question or commenting. The bot needed to actively listen throughout the conversation, not just after finishing each phrase.
- Conversion rate was strongly impacted by the time of day and day of the week.


## Tech stack
What we used for development:
1. Python for ML part:
    - TF-IDF text representation
    - Naive Bayes models
    - regexp where possible 
2. Redis to store all our data. 
3. JSON file for dialogue management
4. Java for all backend operations including dialing and redirecting calls.
5. Third party paid API for non-english speach-to-text


## Conclusion and final thoughts: 
Since the POC was successfully presented and the project received the green light and budget, we were able to 
implement more complex scenarios, expand the scripts, and even bring in a professional actress to record the
voiceovers!   
While these improvements came later, the POC stage was a valuable learning experience.
We realized that machine learning challenges can be solved not only through complex models but 
also by thorough careful domain exploration, data collection, and continuous collaboration within the team.


Thanks for reading!

