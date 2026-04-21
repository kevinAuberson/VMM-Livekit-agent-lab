AI agent for LiveKit SFU
========================

This is a short but fun lab to explore the use of AI agents together with the LiveKit SFU. The goal is to deploy a simple voice agent you can talk to through a WebRTC connection.

LiveKit provides many [agent examples](https://github.com/livekit-examples/python-agents-examples). We will use the simplest one, the [Listen and Respond agent](https://github.com/livekit-examples/python-agents-examples/tree/main/docs/examples/listen_and_respond).

Agent code
----------

Download the file `listen-and-respond.py` from the above link and save it in this directory. It contains the code of the agent that we will deploy.

The agent uses four AI models:

- **Speech-to-text**: this model transcribes the incoming audio (your voice) into text. It uses deepgram/nova-3-general as model.
- **Large language model**: this model generates a text response to the transcribed input. It uses the openai/gpt-4.1-mini model.
- **Text-to-speech**: this model converts the generated text response back into audio. It uses the cartesia/sonic-3 model.
- **Voice activity detection**: this model detects when the user has stopped speaking, so that the agent can start processing the input and generate a response. It uses the silero/vad-3.0 model.

The original code uses the `inference` module which assumes that LiveKit cloud is used to run the SFU and AI models. This is not the case in our lab, since we are running the SFU locally. You therefore need to replace the calls to the `inference` module with direct calls to the AI models:

- Replace the imports at the top of the file with imports of the AI models:

  ```python
  from livekit.agents import JobContext, JobProcess, Agent, AgentSession, AgentServer, cli
  from livekit.plugins import silero, deepgram, openai, cartesia
  ```

- Replace the calls to the `inference` module with direct calls to the models:

  ```python
    ...
    session = AgentSession(
        stt=deepgram.STT(model="nova-3-general"),
        llm=openai.LLM(model="gpt-4.1-mini"),
        tts=cartesia.TTS(model="sonic-3", voice="9626c31c-bec5-4cca-baa8-f8ba9e84c8bc"),
        vad=ctx.proc.userdata["vad"],
        preemptive_generation=True,
    )
    ...
  ```

Environment file
----------------

An environment file `.env` is needed to provide the necessary configuration parameters to the agent.

Create a file named `.env` in this directory with the following content:

```bash
LIVEKIT_API_KEY=<your_api_key>
LIVEKIT_API_SECRET=<your_api_secret>
LIVEKIT_URL=wss://livekit.example.com

# Provider keys (add as needed for specific examples)
OPENAI_API_KEY=<your_openai_api_key>
DEEPGRAM_API_KEY=<your_deepgram_api_key>
CARTESIA_API_KEY=<your_cartesia_api_key>
```

For the LiveKit parameters, use the values of your previously deployed LiveKit SFU. In particular, replace the value of the `LIVEKIT_URL` variable.

Your course instructor will provide you the API keys for the AI models. 

Install the dependencies
------------------------

The easy and modern way to install the dependencies and run the agent is with [Astral UV](https://docs.astral.sh/uv/). If it's not already installed, follow the uv documentation to install it.

Then, run the following command in this directory to install the dependencies and run the agent:

```bash
# Initialize the project and create a virtual environment
uv init
# Install the dependencies
uv add python-dotenv livekit-agents[silero] livekit-plugins-deepgram livekit-plugins-openai livekit-plugins-cartesia
```

Run and test the agent
----------------------

```bash
# Run the agent
uv run python listen-and-respond.py start
```

The command will start the agent and connect it to the LiveKit SFU. It should print a log message indicating that the agent has registered successfully, similar to the following:

```
...
{"message": "registered worker", "level": "INFO", "name": "livekit.agents", "agent_name": "", "id": "AW_xxxxxxxxxxxx", "url": "wss://livekit.example.com", "region": "", "protocol": 17, "timestamp": "2026-04-21T12:00:45.749686+00:00"}
...
```

The agent watches for new rooms being created on the LiveKit SFU and acts as a voice assistant.

Using LiveKit Meet application you previously deployed, connect to the SFU and create a new room. The agent should automatically join the room and start listening to your voice. You can talk to the agent and it will respond with a generated answer.

If something goes wrong, check the logs of the agent for error messages.

Conclusion
----------

No report is required for this lab. It simply demonstrates how to deploy an AI agent together with the LiveKit SFU.

LiveKit provides many other [agent examples for python](https://github.com/livekit-examples/python-agents-examples) or other languages. It is worth exploring them to see the different use cases.

**WebRTC and AI are the future!**
