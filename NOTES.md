# Test notes

## Half-page plan (first 20 minutes, no code)
In the Streamlit UI the user can send a small text (about 4000 characters) and the LLM should print on the screen a summary of that text in 1 sentence as a Vampire .
This is a small backend and a frontend application that interact with each other 
The backend should be capable of handling basic RESTful API requests and the backend should interact with an LLM hosted on Azure.
The frontend should display the data processed by the backend.
Both components should then be deployed to Azure. 

-user experience  in the frontend: he should see on the screen instructions of what he should send in the text box, then insert his text and get a reply on the UI with a summary of one sentence . 
-scalability of the backend : if Azure doesn't answer in 30 seconds, it will stop waiting. gunicorn runs 2 workers, so 2 requests are handled at the same
time
-error handling : be able to get a message and identify it is not empty or too long and to show a clear message on the screen regarding it.
-security measures : all keys and endpoint  secrets are in the .env file ( that is not deployed to GitHub) or on Azure in the App Service settings of the backend app. The input should not be more than about 4000 characters.
### The problem in one sentence
Summarizing a small text into one sentence of a Vampire. 

### What "done" looks like
The user can write a text on Streamlit page , get a 1 sentence summary back. If the text the user sent is empty or too long it will show a clear message on the screen.The apps will run on Azure.

### Three things I'm deliberately not building
- DWH or source data .
- no login system
- not saving history of the chat.
- Do it in English only.

### Where the data comes from
The data that the model was trained on , no extra data should added during this project.

## Decisions and why
- Stack: FastAPI because it is fast and easy way to test API, Streamlit I like it as UI and it is easy to install, gpt-5.4-mini on Azure OpenAI because it is cheap to use and not going to be decommissioned soon
- Architecture: frontend and backend are separate from each other , that way it is easy to debug and change only the definition of one of them ( like different LLM model should  be changed in llm.py).
- Security (where the key lives):all endpoint and keys are in .env (which is in .gitignore), the config.py file reads it from there. on Azure it will be on App Service settings of the backend.
- Deployment:On Azure two Azure App Services

## Problems I hit and how I solved them
-The first prompt was wrong because I forgot to save the main.py file.
-I didn't finish all deployment to Azure in the 2 hours time. I only deployed the backend but not the frontend.

## If I had more time
-test prompt injection
-handle the empty answer instead of showing a blank bubble
-add a test that checks the output is really one sentence
-use more than one example text
-the frontend gives up at 60s while the backend can take 90s, so the user gets a wrong error message and a worker stays busy for nothing. I would make the frontend wait longer than the backend, or reduce the retries.


## Answers to the review questions
1. Time. Where did the 2 hours go? When did the plan finish, when did the first working call happen, when did you start deploying. What would you change in the order?
I started with copy the repo of the template I have and clone it locally.I was busy making sure that the template is working and frontend and the backend are connecting and that the LLM is working . Then I started thinking on the idea and write the Half-page plan  - all of this took around 30 min.
after that I went through all the other components and looking at the code to match the project topic. I wanted to make sure everything is working then making sure all documentation is correct ( like changing in the README file the Environment description ).I took time to test things and changing the code to fix issue I saw on the UI.
Then at the last 20 min I started deploying to Azure, I deployed the backend and started with the frontend , but didn't finish it ( I just had to write 2 more commands) 
Next time I would not start with the template installation, and deploy the empty app first and not leave it to the end .I would start break the problem into small steps and then start building the code small : text in, LLM call, result out.
2. The half-page. Much of it repeats the assignment text. What in it is your own decision? If an interviewer asks "why this problem?", what do you answer?
These are the specific lines that are mine:
-the task itself: summarize into exactly one sentence
-the vampire tone
-the 4000 character limit, and that the user is told about it
-what I chose not to build: no login, no history, English only, no external data
-the decision to invent my own test text rather than use a dataset

I chose this problem because it's a small, testable problem (the output is short enough to check by eye, the failure cases (empty, too long, empty answer) are easy to trigger, and the whole thing runs without any data setup) and the interesting part is the prompt design and the error handling, not the feature list. 
3. Data. You wrote the data is "what the model was trained on". What text did you actually test with? How many examples? How do you know the answer is really one sentence, and really a vampire?
These are the 3 tests I manually did in the UI :
-I tested it with a complaint text that Claude created for me .
-I tested when I send the same text twice (so it is too long)
-I tested what happens when the text is empty with spaces.
All of them use 1 text in 3 ways. I chose a customer complaint because it is long and realistic ( so it is easy to see it will change to a Vampire tone)
It was easy for me to look at the answer and see that it is in a vampire tone and one sentence. I didn't create any test for that on the code. checking by eye works for one example, but it doesn't scale and it isn't repeatable.


4. Errors. You check for empty and too long input. What does the user see if Azure is slow, down, or blocks the text with its content filter? What happens if the model returns an empty answer?
you can see it in the backend\llm.py file:
slow: "The AI service took too long to answer."
down : "Could not reach the AI service."
content filter (400) : "The AI service returned an error."
empty answer : nothing at all, an empty bubble - this is not good for the user experience so this should get fixed.

5. Security. Your key is safe, good. But your backend URL is public. Who can call it, and who pays for that? And what happens if the user's text says "ignore your instructions and do X"?
if my backend is public everyone can call it and that knows the URL. there's no authentication .
I paid for it because I created it under my Azure account.the App Service plan runs by the hour, and every call that reaches Azure OpenAI spends tokens from my deployment. So at the end of the assignment I deleted the App Service.
If the user send overwrite instructions  - it is not handled and I also didn't tested it because I didn't think about it or didn't had time , I should have add it to the system prompt to ignore this kind of things .

6. Scalability. A timeout and 2 workers are settings, not a scaling plan. What happens with 50 users at the same time? What limit on the Azure OpenAI side would you hit first?
With 50 users at once: 2 workers means 2 requests are handled at a time, and the other 48 wait in a queue.
How long they wait: each request waits for Azure, a few seconds normally, but up to 90 seconds in the worst case (30 second timeout, 2 retries). The frontend gives up at 60 seconds, so some users get an error while the backend is still working.
Which limit breaks first: my own app. 
The Azure limit if I scaled up: my model deployment has a tokens-per-minute quota, and when it's exceeded Azure returns 429. My llm.py already turns that into "The AI service is busy." My inputs are long (up to 4000 characters), so the quota would be reached sooner than with short prompts.
What I would do: more workers, scale out to more instances of the app, raise the deployment quota, and add a queue so users are told to wait instead of watching a spinner.


7. Decisions. "I like Streamlit" is true, but it is not a reason an interviewer accepts. Rewrite each decision as: what I chose, what else I could choose, why this one.
I chose Streamlit because:
-The UI is plain Python, so no HTML, CSS or JavaScript, and no separate build step.
-A chat interface is a few lines: st.chat_input, st.chat_message, st.spinner.
-I worked with it in the past.
-it reruns the whole script on every interaction, so you need session_state.
I could have built Gradio UI because I need to build an interface to a model, not a dashboard, and I wanted something quick.I didn't use it because I know Streamlit better and didn't want to spend more time on Gradio.

I chose FastAPI because I had to use RESTful API and it also gives me API test page for free to saved time during the test. I could use Flask(smaller, but validation and API docs need extra libraries)

I chose gpt-5.4-mini because it's cheap, fast enough for one-sentence summaries. A bigger model costs more and is slower for no gain on this task. 
Alternatives: gpt-4.1-mini, or a larger GPT-5 model.


I chose two App Services sharing one B1 plan because the assignment asks for a backend and a frontend that interact; the key stays only in the backend; and each part can be deployed or scaled on its own.
Trade-off: two deployments instead of one, so more steps and more time. Sharing a plan keeps it to one machine and one cost.