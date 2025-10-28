A powerful, real-time voice assistant built with Python that listens for a wake word ("Jarvis") and executes system commands using natural language. Powered by the Qwen3:4B large language model via Ollama and LangChain, it supports speech recognition, text-to-speech, and intelligent command parsing with fallback keyword matching.

Features

Wake Word Detection: Uses Porcupine by Picovoice to detect "Jarvis" with low latency.
Speech Recognition: Real-time transcription using Google Speech Recognition API.
Natural Language Understanding: Qwen3:4B LLM processes commands and returns structured JSON actions.
Text-to-Speech: High-quality voice output using Coqui TTS (XTTS v2) with GPU acceleration (CUDA).
Command Execution:

Open Chrome (open chrome, open google chrome)
Shutdown PC (shutdown, shut down the pc)
Lock laptop, open Notepad, tell time, search web, create folders/files
Sleep mode (sleep) and stop (exit, goodbye)


Fallback Logic: Keyword-based execution if LLM fails or returns invalid JSON.
Robust Error Handling: Retries, timeouts, and detailed logging.
Configurable Start Mode: Normal or sleep mode via jarvis_config.json.


Technologies Used
ComponentTechnologyLanguagePython 3.11LLMQwen3:4B (via Ollama)FrameworkLangChainSpeech RecognitionSpeechRecognition + PyAudioText-to-SpeechCoqui TTS (XTTS v2)Wake WordPicovoice PorcupineAudio RecordingpvrecorderGPU AccelerationPyTorch (CUDA)LoggingBuilt-in logging moduleOS Interactionsubprocess, webbrowser, os

System Requirements

OS: Windows 10/11 (Linux/macOS with minor adjustments)
RAM: 16GB+ recommended (Qwen3:4B uses ~3.5GB VRAM/RAM)
GPU: NVIDIA GPU with CUDA support (RTX 3050 or better)
Storage: 5GB+ free (for model and dependencies)
Internet: Required for Google Speech API and initial model download
