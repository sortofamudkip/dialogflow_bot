## Progress report: chatbot 

[Documentation link](http://dialogflow-python-client-v2.readthedocs.io/en/latest/?#using-dialogflow)

## Brief rundown of `dialogflow`

All of this is done on the Dialogflow website, not locally.

### Training phrases and intents

* `Dialogflow` takes in **input strings**, matches them with **intents**, then responds accordingly.
* `Dialogflow` uses **training phrases** to help decide which intent will be used.
* **Entities** are categories of words in training phrases that improve the accuracy of the matching.

A brief example might be a bot that asks for your favourite colour. The bot first asks "What is your favourite colour?", and has the following training phrases for the *colour* intent:

* <span style="font-family:consolas">My favourite colour is red.</span>
* <span style="font-family:consolas">I really like blue.</span>
* <span style="font-family:consolas">Green has always been a nice colour.</span>

### Entities and fallback intents

Entities are *categories* that help with matching the correct intent. You can choose words in your training phrases to place in entities. You can also use these entities in your response.

To improve the accuracy of the bot, you can give `red`, `blue`, and `green`  the colour entity.

* <span style="font-family:consolas">My favourite colour is <span style="color:blue">red</span>.</span>
* <span style="font-family:consolas">I really like <span style="color:blue">blue</span>.</span>
* <span style="font-family:consolas"><span style="color:blue">Green</span> has always been a nice colour.</span>

When responding, you can use `colour` to refer to the colour they initially said.

The colour entity now has the entries red, blue, and green. You can add **synonyms** for each entry (i.e. red} maroon},ruby},vermillion).

Apart from this intent, we also have the **default** intent, which runs when you start the bot, and the **fallback** intent, when the bot cannot match to any of the existing intents.

You can also create **followup intents** that will match only when a previous intent was matched. For example, the colour-followup intent could ask <b><span style="font-family:consolas">"Why is colour your favourite colour?"</span></b>. 

### Contexts

**Contexts** store information from other intents, so the parameters can be used in other intents.

When B is an **input context** of A, that means B will only be matched when A was matched before. (Think of it like A -> B.)

When D is an **output context** of C, that means when C is matched, it sends its parameters (entities) to D so D can use them later (Also think of it like D  -> C.)

In followup intents (i.e. colour (E)  -> colour-followup (F)), E is automatically an input context of F, and F is also automatically an output context of E.

### Example

Another example is a song player. We have 3 intents, song, turn_on_song, and turn_off_song. The first one simply tells you which song is playing. 

First, you call turn_on_song with a phrase like "Play Africa by Toto". (In this case, Africa would be a Song entity and Toto an Artist entity.) If you want to know what song you're currently playing, match with the song intent. The song name, however, is in turn_on_song, so we can add song to turn_on_song's **output intent** to send the parameters over.

As for turn_off_song, which we call by "Turn it off", this should only be matched if  turn_on_song was called earlier. So turn_on_song becomes the input context of turn_off_song.

## Using this bot

Note: You cannot use the bot locally. It is on Google's API. Thus, you must create the bot **first**, then write a Python script to interact with it.

0. Create a `virtualenv`. (Optional)
1. `pip install dialogflow`
2. Register the chatbot [here](https://console.cloud.google.com/flows/enableapi?apiid=dialogflow.googleapis.com). Optionally, you also need to [enable billing](https://cloud.google.com/billing/docs/how-to/modify-project?visit_id=1-636688875113458644-1673384981&rd=1#enable-billing) if you want to use some of the functions later.
3. Follow [these](https://cloud.google.com/docs/authentication/getting-started) instructions to set up authentication. (If you don't do this, you cannot access the bot at all.) You should recieve a `json` file.

### Current Bot

The current bot can differentiate between the three intents:

* Submitting additional information
* Submitting defect report
* Requesting daily report

### Script

This script is taken mostly from the documentation example. It simply reads in **user commands** and tells you what the **intent** is.

```python=
import dialogflow_v2 as dialogflow
import os

os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "path\to\file\from\step_3"

def detect_intent_texts(project_id, session_id, texts, language_code):
    session_client = dialogflow.SessionsClient()

    session = session_client.session_path(project_id, session_id)

    for text in texts:
        text_input = dialogflow.types.TextInput(
            text=text, language_code=language_code)

        query_input = dialogflow.types.QueryInput(text=text_input)

        response = session_client.detect_intent(
            session=session, query_input=query_input)

        print('Query text: {}'.format(response.query_result.query_text))
        print('Detected intent: {} (confidence: {})'.format(
            response.query_result.intent.display_name,
            response.query_result.intent_detection_confidence))
        print('Fulfillment text: {}\n'.format(
            response.query_result.fulfillment_text))


while True:
	print("Input a phrase:", end=" ")
	a = input()
	detect_intent_texts("task-manager-274d7", "abcde", [a], "en-US")
```

### Demo

```
>python agent.py
Input a phrase: hi
Query text: hi
Detected intent: Default Welcome Intent (confidence: 1.0)
Fulfillment text: Hello! How I can help you?

Input a phrase: I found a bug
Query text: I found a bug
Detected intent: dev_defect (confidence: 1.0)
Fulfillment text: You wanted to submit a defect report. Is that correct?

Input a phrase: I want to submit a report
Query text: I want to submit a report
Detected intent: report (confidence: 1.0)
Fulfillment text: You wanted to add some progress info. Is that correct?

Input a phrase: I have some notes
Query text: I have some notes
Detected intent: report (confidence: 0.7300000190734863)
Fulfillment text: You wanted to add some progress info. Is that correct?


Input a phrase: I want the daily report
Query text: I want the daily report
Detected intent: leader_daily_report (confidence: 1.0)
Fulfillment text: You, the dev leader, wanted the daily report. Am I correct?

Input a phrase: dev notes    //note: this one is incorrect!
Query text: dev notes
Detected intent: leader_daily_report (confidence: 0.8199999928474426)
Fulfillment text: You, the dev leader, wanted the daily report. Am I correct?
```

### Results

The script can connect to the bot and exchange information.

The bot produces correct results most of the time, but its speed varies quite a bit. It is sometimes so slow that the connection will time out, causing the program to end.
