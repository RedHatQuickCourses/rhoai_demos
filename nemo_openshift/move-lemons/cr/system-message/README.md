## Deployment

Set environment variables with your LLM credentials:
```bash
export LLM_API_BASE="https://your-llm-endpoint.com/v1"
export LLM_MODEL_NAME="your-model-name"
export LLM_API_KEY="your-api-key"
```

Deploy with envsubst:
```bash
envsubst '${LLM_API_KEY}' < secret.yaml | oc apply -f -
envsubst '${LLM_API_BASE} ${LLM_MODEL_NAME}' < configmap.yaml | oc apply -f -
oc apply -f nemo.yaml
```

## Testing

Get the route URL:
```bash
NEMO_ROUTE=$(oc get route lemonade -o jsonpath='{.spec.host}')
echo "NeMo Guardrails URL: https://$NEMO_ROUTE"
```

### Test 1: Basic Lemon Question (Should Work)

```bash
curl -s -X POST "https://$NEMO_ROUTE/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$LLM_MODEL_NAME"'",
    "messages": [{"role":"user","content":"What are the health benefits of lemons?"}]
  }' | jq
```

should return something along the lines of:

```json
{
  "id": "chatcmpl-92e0f92c-f312-4814-87dc-6e3ea705b6a1",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "Lemons are rich in vitamin C, which is essential for boosting the immune system and helping the body absorb iron. The vitamin C content may also aid in promoting skin health and reducing the risk of certain chronic diseases. Lemons contain antioxidants, which help protect the body from oxidative stress and may reduce inflammation. The dietary fiber found in lemons supports healthy digestion. Additionally, the citric acid in lemons may enhance metabolism and aid in weight management. Studies suggest that lemons may have potential benefits in managing cholesterol levels and reducing kidney stone risk by increasing urinary citrate levels. Some research also implies they may help with blood pressure regulation and reduction of free radical damage. Lemons are often used in detox and cleansing diets, although scientific evidence on these claims varies. Always consider the acidity of lemon juice and its effect on tooth enamel. As with any food, it's best to consume lemons as part of a balanced diet.",
        "role": "assistant"
      }
    }
  ],
  "created": 1772218174,
  "model": "microsoft/phi-4",
  "object": "chat.completion",
  "guardrails": {
    "config_id": "lemonade-stand"
  }
}
```

### Test 2: Off-Topic Question (Should Refuse)

```bash
curl -s -X POST "https://$NEMO_ROUTE/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$LLM_MODEL_NAME"'",
    "messages": [{"role":"user","content":"Tell me about oranges"}]
  }' | jq
```

should return something along the lines of:

```json
{
  "id": "chatcmpl-ed2d32bc-aa0a-49da-813c-ae3cb7077974",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "I'm sorry, but I can only provide information about lemons. If you'd like to know about the cultivation, varieties, storage, preparation, uses, safety, history, science, or lemon-based recipes, feel free to ask about those topics!",
        "role": "assistant"
      }
    }
  ],
  "created": 1772218193,
  "model": "microsoft/phi-4",
  "object": "chat.completion",
  "guardrails": {
    "config_id": "lemonade-stand"
  }
}
```

### Test 3: Streaming Response

```bash
curl -s -X POST "https://$NEMO_ROUTE/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$LLM_MODEL_NAME"'",
    "messages": [{"role":"user","content":"Give me a recipe for lemonade"}],
    "stream": true
  }'
```

should return something along the lines of

```json
data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "Certa"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "i"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "nly! "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "He"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "re"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "'s a b"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "asic "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "idea"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " for ma"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ki"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ng a refres"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "hing l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emon beve"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "rag"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "e"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ":"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "\n\n1"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ". **Ingredi"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "en"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ts"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "**"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ":\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218206, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - F"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "re"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "shly sque"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ezed l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emon j"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "u"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ic"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "e\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - W"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "a"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "te"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "r\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - S"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "weet"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "en"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "er ("}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "suc"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "h as s"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "uga"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "r or h"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "on"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ey"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ")\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   -"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " Ice c"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218207, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "u"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "be"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "s\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - Opti"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "o"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "nal: "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "Mint le"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "aves"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " for "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "gar"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "nis"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "h or sl"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ice"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "s of l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emon"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " for decora"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ti"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "o"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "\n\n2"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ". **Instruct"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "io"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ns"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "**"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ":\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218208, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "- "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "In a pit"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "c"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "her, com"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "b"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "i"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ne 1"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " cu"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "p of fre"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "shly sque"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ezed l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emon j"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "uice "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "w"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "i"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "th 1"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " cu"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "p of "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "your ch"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "osen s"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "weet"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "e"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ner, stir"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ring u"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218209, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ntil comple"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "tely disso"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "lv"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ed"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ".\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "Gradu"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ally"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " add a"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "b"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "o"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ut "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "3"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "to 4 "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "cup"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "s of "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "cold w"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "a"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ter, adjus"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "tin"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "g to "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "your t"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "aste"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " for l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218210, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emon inten"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "sity"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " and sweet"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ne"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ss"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ".\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "Stir "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "well"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " and s"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "erve "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "over"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " i"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ce"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": ".\n"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "   - "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "Gar"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "nish "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "with "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "mint le"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ave"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "s or sl"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ice"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "s of l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emo"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218211, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "n if des"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ire"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "d.\n\nE"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "njoy "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "your refres"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "hing l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "emon-b"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ased beve"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "r"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "age!"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " Le"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "t me "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "kno"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "w if"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": " you "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "need "}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "more informa"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "tion a"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "bou"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "t le"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "mons themse"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "l"}, "index": 0, "finish_reason": null}]}

data: {"id": "chatcmpl-448adae1-43b2-48a0-975b-46925e90fbcf", "object": "chat.completion.chunk", "created": 1772218212, "model": "microsoft/phi-4", "choices": [{"delta": {"content": "ves."}, "index": 0, "finish_reason": null}]}

data: [DONE]
```

