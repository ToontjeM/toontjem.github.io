# How to Use Hugging Face Models with Ollama
### Example

Model is LLama-3.1-8B-Lexi-Uncensored-V2-GGUF.
- Go to (https://huggingface.co/Orenguteng/Llama-3.1-8B-Lexi-Uncensored-V2-GGUF)
- Download one of the GGUF model files to your computer. The bigger the higher quality and uses more resources.
- Click on `Files and Versions` on the model page
- Save the file
- Open a terminal where you put that file and create a Modelfile using `vi Modelfile`
- Add a FROM and SYSTEM section to the file. The FROM points to your model file you just downloaded, and the SYSTEM prompt is the core model instructions for it to follow on every request.
- Create a new LLM using `ollama create lexiwriter`
- Run your new model.
