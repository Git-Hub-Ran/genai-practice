# Test notes

## Half-page plan (first 20 minutes, no code)
In the Streamlit UI the user can send a small text (about 4000 characters) and the LLM should print on the screen a summary of that text in 1 sentance as a Vampire .
This is a small backend and a frontend application that interact with each other 
The backend should be capable of handling basic RESTful API requests and the backend should interact with an LLM hosted on Azure.
The frontend should display the data processed by the backend.
Both components should then be deployed to Azure. 

-user experience  in the frontend: he should see on the screen instructions of what he should send in the text box, then insert his text and get a reply on the UI with a summery of one sentance . 
-scalability of the backend : if Azure doesn't answer in 30 seconds, it will stop waiting. gunicorn runs 2 workers, so 2 requests are handled at the same
time
-error handling : be able to get amessage and identify it is not empty or too long and to show a clear message on the screen regarding it.
-security measures : all keys and endpoint  secreats are in the .env file ( that is not deployed to GIt hub) or on Azure in the App Service settings of the backend app. The input shoudl not be more then about 4000 characters.
### The problem in one sentence
Summarizing a small text into one sentence of a Vampire. 

### What "done" looks like
The user can write a text on Sreamlit page , get a 1 sentanc summary back. If the txt the user sent is empty or too long it will show a clear message on the screen.The apps will run on Azure.

### Three things I'm deliberately not building
- DWH or source data .
- no login system
- not saving history of the chat.
- Do it in English only.

### Where the data comes from
The data that the model was trained on , no extra data should added during this project.

## Decisions and why
- Stack: FastAPI because it is fast and easy way to test API, Streamlit I like it as UI and it is easy to install, gpt-5.4-mini on Azure OpenAI because it is cheap to use and not going to demeniss soon
- Architecture: frontend and backend are seperate from each other , that way it is easy to debug and change only the definition of one of them ( like diffrent LLM model shoudl  be change in llm.py).
- Security (where the key lives):all endpoint and keys are in .env (which is in .gitignore), the config.py file reads it from there. on Azure it will be on App Service settings of the backend.
- Deployment:On Azure two Azure App Services

## Problems I hit and how I solved them
-The first prompt was wrong because I forgot to save the main.py file.
-I didn't finish all deployment to Azure in the 2 hours time. I only deployed the backend but not the frontend.

## If I had more time
-






1. Time. Where did the 2 hours go? When did the plan finish, when did the first working call happen, when did you start deploying. What would you change in the order?

2. The half-page. Much of it repeats the assignment text. What in it is your own decision? If an interviewer asks "why this problem?", what do you answer?

3. Data. You wrote the data is "what the model was trained on". What text did you actually test with? How many examples? How do you know the answer is really one sentence, and really a vampire?
I tested it with a complaint text that Claude created for me .
I tested when I send the smae text twice ( so it is too long)
I tested what heppen when the text is empty with spaces


4. Errors. You check for empty and too long input. What does the user see if Azure is slow, down, or blocks the text with its content filter? What happens if the model returns an empty answer?

5. Security. Your key is safe, good. But your backend URL is public. Who can call it, and who pays for that? And what happens if the user's text says "ignore your instructions and do X"?

6. Scalability. A timeout and 2 workers are settings, not a scaling plan. What happens with 50 users at the same time? What limit on the Azure OpenAI side would you hit first?

7. Decisions. "I like Streamlit" is true, but it is not a reason an interviewer accepts. Rewrite each decision as: what I chose, what else I could choose, why this one.
8. Grammer : Can I ask Claude at teh end just to fix my spelling mostakes?