# Ollama

## Store mode in external hardisk

<https://github.com/ollama/ollama/issues/2322>

```bash
export OLLAMA_MODELS=/path/to/my/disk/models
./ollama serve
```


## Resume download

<https://github.com/ollama/ollama/issues/695>

```bash
export OLLAMA_NOPRUNE=true ollama serve
```

## Deepseek

<https://ollama.com/library/deepseek-r1:1.5b>

1.1 GB


```bash
ollama run deepseek-r1:1.5b
```