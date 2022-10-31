# Rasa lab

Rasa is a data-driven conversational framework which means that it is "data first". You will define a training set, extend it through 'mock' interactions, train a model, interact and repeat. This is in contrast to the Furhat lab where each dialog state had well defined rules, i.e. an explicit conversational flow, moving from state to state, whereas Rasa utilizes a machine learning approach where an agent, trained on your dataset, decides what actions to take.

In this tutorial we will install Rasa, initialize a new project and make slight modifications from the original Rasa playground example. The steps involve:
- Create python environment
- Install rasa
- Create a project with `rasa init`
- Go to [Playground](https://rasa.com/docs/rasa/playground)
  - Read the steps in 'build your assistant' section
  - But instead of downloading the project directly (it seems to contain deprecated syntax) we will make slight modifications ourselves.
- Overview of rasa init project
  - Look in `domain.yml`
    - This file defines all the "things" your agent "knows"
    - What intents and responses does it use?
  - Look inside your project in the `data/` folder to see the training data.
    - `data/nlu.yaml` stores your nlu data samples
    - `data/stories.yaml` stores your conversations (e.g. intent->action->intent->action)
    - `data/rules.yaml` hardcoded rules for specific states (should be kept minimal)
  - `config.yml` contains the configuration of the NLP systems used (can be left unchanged, many comments by default).


## Extend the rasa-init-project to recover the playground-example
If you train and download directly from the playground the format/syntax/versioning seems to be off so we use rasa init and rebuild the playground-example.

#### For this example we want to:
1. Add a special entity called `email`
2. Add responses the agent will use. (in `domain.yml`)
3. Add a form (slot-filling)  (in `domain.yml`)
4. Add basic rules for the form (in `data/rules.yml`)
5. Copy sample data and update `data/nlu.yml`, `data/stories.yml`, `domain.yml`
    - intents, entities, stories, responses


#### Todo
* Entity: define an entity `email`. (in `domain.yml`)
    ```yaml
    entities:
      - email
    ```
* Responses: copy utterances `utter_ask_email`, `utter_subscribed` from playground. (in `domain.yml`)
  ```yaml
  responses:
    utter_ask_email:
    - text: What is your email address?
    utter_subscribed:
    - text: Check your inbox at {email} in order to finish subscribing to the newsletter!
    - text: You're all set! Check your inbox at {email} to confirm your subscription.
  ```
* Intents: add the new playground intents `subscribe` and `inform` to `domain.yml`.
  ```yaml
  intents:
    - greet
    ...
    - subscribe
    - inform
  ```
* Copy the form (slots and form) from playground but change mappings `- type: from_text` -> `- type: from_entity`
  ```yaml
  # Add Slots and Forms (i.e. slot-filling)
  slots:
    email:
      type: text
      mappings:
      - type: from_entity  # NEW instead of filling the slot directly from text we use an entity
        entity: email      # NEW the entity to use
        conditions:
        - active_loop: newsletter_form
          requested_slot: email
  forms:
    newsletter_form:
      required_slots:
      - email
  ```
* Copy the `rules` from playground to the `data/rules.yml` file. [Rules Docs](https://rasa.com/docs/rasa/training-data-format/#rules)
  ```yaml
  # Add rules for newsletter_form
  - rule: activate subscribe form
    steps:
    - intent: subscribe
    - action: newsletter_form
    - active_loop: newsletter_form

  - rule: submit form
    condition:
    - active_loop: newsletter_form
    steps:
    - action: newsletter_form
    - active_loop: null
    - action: utter_subscribed
  ```
* Data
  * Intents: Copy the intents datapoints from playground to `data/nlu.yml`.
    * Copy the 'subscribe' and 'inform' intents. [Intent Docs](https://rasa.com/docs/rasa/training-data-format/#entities)
    * Annotate special email entities `[example@example.com](email)`. [Entities Docs](https://rasa.com/docs/rasa/training-data-format/#entities)
  * Stories: add the `story` from the playground example to `data/stories.yml`. [Stories Docs](https://rasa.com/docs/rasa/training-data-format/#stories)
    ```yaml
    - story: greet and subscribe
      steps:
      - intent: greet
      - action: utter_greet
      - intent: subscribe
      - action: newsletter_form
      - active_loop: newsletter_form
    ```

#### Train, interact, debug, collect more data
* Train the model
    * `rasa train`
    * Be wary of information of `intents`, `entities` and make sure everything is defined in `domain.yml`
    * Look in your `config.yml` file to get a sense of the NLP/AI modules used
* Converse with your model
    * `rasa shell`
    * Try exact emails as provided in the training data (e.g. "random@example.com")
    * Try a different email, does it work? (e.g. "my email is myself@kth.se")
    * Keep calm and move to next point.
* Add more data to your training set in a controlled way
    1. Add more training samples for specific `intents` directly in `data/nlu.yml`
      * Add ten more datapoints to the `inform` intent using various different email adresses.
      * train the model again and interact, did it catch the email entities?
    2. Use: `rasa interactive`
      * In interactive mode you can see what your model thinks is going on, correct its mistakes and save new data
      * add some new conversations, 'export and quit', look in your dataset for changes and retrain the model.
* Debug the model by running: `rasa shell --debug`
