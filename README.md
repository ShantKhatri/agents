<!--BEGIN_BANNER_IMAGE-->
<p align="center">
  <img src="https://assets.videosdk.live/images/github-banner.png" alt="VideoSDK AI Agents Banner" style="width:100%;">
</p>
<!--END_BANNER_IMAGE-->

<h1 align="center">VideoSDK AI Agents</h1>

<p align="center">
  Open-source Python framework for building production-ready, real-time voice and multimodal AI agents.
</p>

<p align="center">
  <a href="https://pypi.org/project/videosdk-agents/"><img src="https://img.shields.io/pypi/v/videosdk-agents" alt="PyPI Version"></a>
  <a href="https://pepy.tech/projects/videosdk-agents"><img src="https://static.pepy.tech/badge/videosdk-agents/month" alt="Downloads"></a>
  <a href="https://x.com/video_sdk"><img src="https://img.shields.io/twitter/follow/video_sdk" alt="Twitter"></a>
  <a href="https://www.youtube.com/c/VideoSDK"><img src="https://img.shields.io/badge/YouTube-VideoSDK-red" alt="YouTube"></a>
  <a href="https://www.linkedin.com/company/video-sdk/"><img src="https://img.shields.io/badge/LinkedIn-VideoSDK-blue" alt="LinkedIn"></a>
  <a href="https://discord.com/invite/f2WsNDN9S5"><img src="https://img.shields.io/badge/Discord-Join%20Us-7289DA" alt="Discord"></a>
  <a href="https://deepwiki.com/videosdk-live/agents"><img src="https://deepwiki.com/badge.svg" alt="Ask DeepWiki"></a>
  <a href="https://github.com/videosdk-live/agents/blob/main/LICENSE.txt"><img src="https://img.shields.io/github/license/videosdk-live/agents" alt="License"></a>
</p>

<br />

<!--BEGIN_DESCRIPTION-->
**VideoSDK AI Agents** is a Python SDK for building AI agents that join VideoSDK rooms as real-time participants. It connects your agent worker, AI models, and user devices into a single low-latency pipeline, handling audio streaming, turn detection, interruptions, and media routing automatically so you can focus on agent logic.
<!--END_DESCRIPTION-->

![VideoSDK AI Agents High Level Architecture](https://assets.videosdk.live/images/agent-architecture.png)


## Features

- **Three Pipeline Modes**: Cascade (STT → LLM → TTS) for full provider control, Realtime for sub-500ms latency, or Hybrid to mix both.
- **20+ Provider Integrations**: OpenAI, Gemini, Anthropic, Deepgram, ElevenLabs, Cartesia, and [many more](#supported-libraries-and-plugins).
- **Pipeline Hooks**: Intercept and transform data at any stage using the `@pipeline.on(...)` decorator, no subclassing required.
- **MCP Integration**: Connect agents to external data sources, internal APIs, and toolchains natively.
- **Agent-to-Agent (A2A)**: Reliable multi-agent routing with correlation-based request tracking for complex handoffs.
- **SIP & Telephony**: Handle inbound and outbound PSTN calls via VideoSDK's built-in SIP support.
- **Virtual Avatars**: Plug in Simli, Anam, or any custom provider. Audio routing, lip-sync, and teardown are handled automatically.
- **Built-in Observability**: OpenTelemetry tracing and structured logging per component, out of the box.


## Installation

```bash
pip install "videosdk-agents[openai,deepgram,cartesia,silero,turn-detector]"
```

You will need a `VIDEOSDK_AUTH_TOKEN` from the [VideoSDK Dashboard](https://app.videosdk.live), plus API keys for your chosen providers.

## Quick Start
 
```python
from videosdk.agents import Agent, AgentSession, WorkerJob, RoomOptions, JobContext
from videosdk.agents import Pipeline
from videosdk.plugins.deepgram import DeepgramSTT
from videosdk.plugins.google import GoogleLLM
from videosdk.plugins.cartesia import CartesiaTTS
from videosdk.plugins.silero import SileroVAD
from videosdk.plugins.namo_turn_detector import TurnDetector
 
class VoiceAgent(Agent):
    def __init__(self):
        super().__init__(
            instructions="You are a helpful voice assistant built by VideoSDK."
        )
 
    async def on_enter(self) -> None:
        await self.session.say("Hi there! How can I help you today?")
 
async def start_session(context: JobContext):
    pipeline = Pipeline(
        stt=DeepgramSTT(),
        llm=GoogleLLM(),
        tts=CartesiaTTS(),
        vad=SileroVAD(),
        turn_detector=TurnDetector(),
    )
    session = AgentSession(agent=VoiceAgent(), pipeline=pipeline)
    await session.start(wait_for_participant=True, run_until_shutdown=True)
 
if __name__ == "__main__":
    job = WorkerJob(
        entrypoint=start_session,
        jobctx=lambda: JobContext(RoomOptions(room_id="<YOUR_MEETING_ID>", playground=True))
    )
    job.start()
```
 
> **Tip:** Set `playground=True` in `RoomOptions` to test locally using your mic and speakers, no VideoSDK room needed.
 
Required environment variables: `VIDEOSDK_AUTH_TOKEN`, `DEEPGRAM_API_KEY`, `GOOGLE_API_KEY`, `CARTESIA_API_KEY`
 

## Pipeline Modes

The unified `Pipeline` class auto-detects the right execution mode based on what you pass in.

| Mode | When to use | Example |
|---|---|---|
| **Cascade** | Full control over STT, LLM, TTS separately | `Pipeline(stt=..., llm=..., tts=...)` |
| **Realtime** | Lowest latency with a unified model | `Pipeline(llm=GeminiRealtime(...))` |
| **Hybrid** | Custom STT with a realtime model, or vice versa | `Pipeline(stt=..., llm=OpenAIRealtime(...))` |

### VideoSDK Inference

Access top-tier models without managing individual API keys. Billed directly through your VideoSDK account.

```python
from videosdk.agents.inference import STT, LLM, TTS, Realtime

async def start_session(context: JobContext):
    pipeline = Pipeline(
        stt=STT.sarvam(model_id="saarika:v2.5", language="en-IN"),
        llm=LLM.google(model_id="gemini-2.5-flash"),
        tts=TTS.sarvam(model_id="bulbul:v2", speaker="anushka", language="en-IN"),
        denoise=Denoise.sanas(),
        vad=SileroVAD(),
    )
    session = AgentSession(agent=MyAgent(), pipeline=pipeline)
    await session.start(wait_for_participant=True, run_until_shutdown=True)
```

## Core Concepts

| Concept | Description |
|---|---|
| **Agent** | Defines the AI's personality, tools, and conversational logic. |
| **AgentSession** | Connects your Agent to a VideoSDK room and manages its lifecycle. |
| **Pipeline** | Wires STT, LLM, and TTS together and selects the optimal execution mode. |
| **JobContext** | Carries room metadata and configuration when starting an agent worker. |


## Supported Libraries and Plugins

| Category | Services |
|---|---|
| **Real-time Models** | [OpenAI](https://docs.videosdk.live/ai_agents/plugins/realtime/openai) · [Gemini](https://docs.videosdk.live/ai_agents/plugins/realtime/google-live-api) · [AWS Nova Sonic](https://docs.videosdk.live/ai_agents/plugins/realtime/aws-nova-sonic) · [Azure Voice Live](https://docs.videosdk.live/ai_agents/plugins/realtime/azure-voice-live) |
| **Speech-to-Text (STT)** | [OpenAI](https://docs.videosdk.live/ai_agents/plugins/stt/openai) · [Google](https://docs.videosdk.live/ai_agents/plugins/stt/google) · [Azure AI Speech](https://docs.videosdk.live/ai_agents/plugins/stt/azure-ai-stt) · [Azure OpenAI](https://docs.videosdk.live/ai_agents/plugins/stt/azureopenai) · [Sarvam AI](https://docs.videosdk.live/ai_agents/plugins/stt/sarvam-ai) · [Deepgram](https://docs.videosdk.live/ai_agents/plugins/stt/deepgram) · [Cartesia](https://docs.videosdk.live/ai_agents/plugins/stt/cartesia-stt) · [AssemblyAI](https://docs.videosdk.live/ai_agents/plugins/stt/assemblyai) · [Navana](https://docs.videosdk.live/ai_agents/plugins/stt/navana) |
| **Language Models (LLM)** | [OpenAI](https://docs.videosdk.live/ai_agents/plugins/llm/openai) · [Azure OpenAI](https://docs.videosdk.live/ai_agents/plugins/llm/azureopenai) · [Google](https://docs.videosdk.live/ai_agents/plugins/llm/google-llm) · [Sarvam AI](https://docs.videosdk.live/ai_agents/plugins/llm/sarvam-ai-llm) · [Anthropic](https://docs.videosdk.live/ai_agents/plugins/llm/anthropic-llm) · [Cerebras](https://docs.videosdk.live/ai_agents/plugins/llm/Cerebras-llm) |
| **Text-to-Speech (TTS)** | [OpenAI](https://docs.videosdk.live/ai_agents/plugins/tts/openai) · [Google](https://docs.videosdk.live/ai_agents/plugins/tts/google-tts) · [AWS Polly](https://docs.videosdk.live/ai_agents/plugins/tts/aws-polly-tts) · [Azure AI Speech](https://docs.videosdk.live/ai_agents/plugins/tts/azure-ai-tts) · [Azure OpenAI](https://docs.videosdk.live/ai_agents/plugins/tts/azureopenai) · [Deepgram](https://docs.videosdk.live/ai_agents/plugins/tts/deepgram) · [Sarvam AI](https://docs.videosdk.live/ai_agents/plugins/tts/sarvam-ai-tts) · [ElevenLabs](https://docs.videosdk.live/ai_agents/plugins/tts/eleven-labs) · [Cartesia](https://docs.videosdk.live/ai_agents/plugins/tts/cartesia-tts) · [Resemble AI](https://docs.videosdk.live/ai_agents/plugins/tts/resemble-ai-tts) · [Smallest AI](https://docs.videosdk.live/ai_agents/plugins/tts/smallestai-tts) · [Speechify](https://docs.videosdk.live/ai_agents/plugins/tts/speechify-tts) · [InWorld](https://docs.videosdk.live/ai_agents/plugins/tts/inworld-ai-tts) · [Neuphonic](https://docs.videosdk.live/ai_agents/plugins/tts/neuphonic-tts) · [Rime AI](https://docs.videosdk.live/ai_agents/plugins/tts/rime-ai-tts) · [Hume AI](https://docs.videosdk.live/ai_agents/plugins/tts/hume-ai-tts) · [Groq](https://docs.videosdk.live/ai_agents/plugins/tts/groq-ai-tts) · [LMNT AI](https://docs.videosdk.live/ai_agents/plugins/tts/lmnt-ai-tts) · [Papla Media](https://docs.videosdk.live/ai_agents/plugins/tts/papla-media) |
| **Voice Activity Detection** | [SileroVAD](https://docs.videosdk.live/ai_agents/plugins/silero-vad) |
| **Turn Detection** | [Namo Turn Detector](https://docs.videosdk.live/ai_agents/plugins/namo-turn-detector) |
| **Virtual Avatar** | [Simli](https://docs.videosdk.live/ai_agents/core-components/avatar) · [Anam](https://docs.videosdk.live/ai_agents/plugins/avatar/anam) |
| **LLM Orchestration** | [LangChain](https://docs.videosdk.live/ai_agents/plugins/llm/langchain) · [LangGraph](https://docs.videosdk.live/ai_agents/plugins/llm/langgraph) |
| **Denoise** | [RNNoise](https://docs.videosdk.live/ai_agents/core-components/de-noise) |

## Examples

<table>
<tr>
<td width="50%">
<h3>🎙️ <a href="examples/cascade_basic.py">Cascade Mode (Basic)</a></h3>
<p>STT → LLM → TTS voice agent using Google, Deepgram, and Cartesia.</p>
</td>
<td width="50%">
<h3>⚡ <a href="examples/realtime_basic.py">Realtime Mode</a></h3>
<p>Minimal realtime agent using Gemini Live for lowest-latency voice interactions.</p>
</td>
</tr>
<tr>
<td width="50%">
<h3>🔀 <a href="examples/hybrid_mode(cascade+realtime)/">Hybrid Mode</a></h3>
<p>Mix cascade and realtime components: custom STT with a realtime LLM.</p>
</td>
<td width="50%">
<h3>🪝 <a href="examples/voice_pipeline_hooks.py">Pipeline Hooks</a></h3>
<p>Intercept and transform data at any stage using <code>@pipeline.on(...)</code>.</p>
</td>
</tr>
<tr>
<td width="50%">
<h3>🤝 <a href="examples/a2a/">Agent-to-Agent (A2A)</a></h3>
<p>Multi-agent workflow: receptionist that hands off to a loan specialist agent.</p>
</td>
<td width="50%">
<h3>👁️ <a href="examples/vision/">Vision Agent</a></h3>
<p>Multimodal agent that processes live video frames alongside voice.</p>
</td>
</tr>
<tr>
<td width="50%">
<h3>🏥 <a href="use_case_examples/appointment_booking_agent.py">Appointment Booking</a></h3>
<p>Production-ready healthcare receptionist for scheduling clinic appointments.</p>
</td>
<td width="50%">
<h3>🌐 <a href="examples/mcp_server_examples/">MCP Server Integration</a></h3>
<p>Stock market analyst agent with real-time financial data via MCP.</p>
</td>
</tr>
</table>

For the full list, see the [`examples`](examples/) directory.


## Running Your Agent

**Terminal (local testing)**

```bash
python myagent.py
```

Set `playground=True` in `RoomOptions` to use your local mic and speakers without an active room.

**With a VideoSDK client**

Generate a token and meeting ID from the [VideoSDK Dashboard](https://app.videosdk.live), then connect using any quickstart client: [JavaScript](https://github.com/videosdk-live/quickstart/tree/main/js-rtc) · [React](https://github.com/videosdk-live/quickstart/tree/main/react-rtc) · [React Native](https://github.com/videosdk-live/quickstart/tree/main/react-native) · [Flutter](https://github.com/videosdk-live/quickstart/tree/main/flutter-rtc) · [iOS](https://github.com/videosdk-live/quickstart/tree/main/ios-rtc) · [Android](https://github.com/videosdk-live/quickstart/tree/main/android-rtc)


## Docs and Guides

Full tutorials, API references, and deployment guides: **[docs.videosdk.live/ai_agents](https://docs.videosdk.live/ai_agents/introduction)**


## Contributing

Contributions are welcome. New plugins, bug fixes, examples, and feedback all help. See the [Plugin Development Guide](BUILD_YOUR_OWN_PLUGIN.md) to integrate a new provider.

**Development setup**

```bash
# Using uv (recommended)
git clone https://github.com/videosdk-live/agents.git
cd agents && uv sync
uv run python examples/cascade_basic.py

# Using pip
git clone https://github.com/videosdk-live/agents.git
cd agents && bash setup.sh && source venv/bin/activate
python examples/cascade_basic.py
```

Join the conversation on [Discord](https://discord.com/invite/f2WsNDN9S5).

---

<!--BEGIN_REPO_NAV-->
<table>
<thead><tr><th colspan="2">VideoSDK Ecosystem</th></tr></thead>
<tbody>
<tr><td>Agents SDKs</td><td><b>Python</b></td></tr>
<tr><td>VideoSDK SDKs</td><td><a href="https://docs.videosdk.live/react/guide/video-and-audio-calling-api-sdk/concept-and-architecture">React</a> · <a href="https://docs.videosdk.live/react-native/guide/video-and-audio-calling-api-sdk/quick-start">React Native</a> · <a href="https://docs.videosdk.live/android/guide/video-and-audio-calling-api-sdk/quick-start">Android</a> · <a href="https://docs.videosdk.live/ios/guide/video-and-audio-calling-api-sdk/quick-start">iOS</a> · <a href="https://docs.videosdk.live/flutter/guide/video-and-audio-calling-api-sdk/quick-start">Flutter</a> · <a href="https://docs.videosdk.live/unity/guide/video-and-audio-calling-api-sdk/quick-start">Unity</a></td></tr>
<tr><td>Starter Apps</td><td><a href="https://github.com/videosdk-community/ai-agent-examples">Python Agent</a> · <a href="https://github.com/videosdk-live/quickstart/tree/main/react-rtc">React App</a> · <a href="https://github.com/videosdk-live/quickstart/tree/main/ios-rtc">SwiftUI App</a> · <a href="https://github.com/videosdk-live/quickstart/tree/main/android-rtc">Android App</a> · <a href="https://github.com/videosdk-live/quickstart/tree/main/flutter-rtc">Flutter App</a> · <a href="https://github.com/videosdk-live/quickstart/tree/main/react-native">React Native App</a></td></tr>
<tr><td>Resources</td><td><a href="https://docs.videosdk.live">Docs</a> · <a href="https://app.videosdk.live">Dashboard</a></td></tr>
<tr><td>Community</td><td><a href="https://discord.com/invite/f2WsNDN9S5">Discord</a> · <a href="https://x.com/video_sdk">X</a> · <a href="https://www.youtube.com/c/VideoSDK">YouTube</a> · <a href="https://www.linkedin.com/company/video-sdk/">LinkedIn</a></td></tr>
</tbody>
</table>
<!--END_REPO_NAV-->

<br />

<p align="center">
  <a href="https://github.com/videosdk-live/agents/graphs/contributors">
    <img src="https://contrib.rocks/image?repo=videosdk-live/agents" />
  </a>
  <br/>
  <strong>Made with ❤️ by The VideoSDK Team</strong>
</p>

## License

VideoSDK AI Agents is open-source, released under the [Apache License 2.0](LICENSE.txt).