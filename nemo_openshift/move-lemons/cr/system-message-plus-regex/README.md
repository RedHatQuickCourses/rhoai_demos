## Deployment

Set environment variables with your LLM credentials:
```bash
export LLM_API_BASE="https://your-llm-endpoint.com/v1"
export LLM_MODEL_NAME="your-model-name"
export LLM_API_KEY="your-api-key"
```

Deploy with envsubst:
```bash
envsubst < secret.yaml | oc apply -f -
envsubst '${LLM_API_BASE} ${LLM_MODEL_NAME} ${LLM_API_KEY}' < configmap.yaml | oc apply -f -
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

### Test 3: Regex Response

```bash
curl -s -X POST "https://$NEMO_ROUTE/v1/guardrail/checks" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$LLM_MODEL_NAME"'",
    "messages": [{"role":"user","content":"My password for bot account is abc123456"}]
  }' | jq
```

```json
{
  "status": "blocked",
  "rails_status": {
    "regex check input": {
      "status": "blocked"
    }
  },
  "messages": [
    {
      "index": 0,
      "role": "user",
      "rails": {
        "regex check input": {
          "status": "blocked"
        }
      }
    }
  ],
  "guardrails_data": {
    "log": {
      "activated_rails": [
        "regex check input"
      ],
      "stats": {
        "input_rails_duration": 0.01584339141845703,
        "dialog_rails_duration": null,
        "generation_rails_duration": null,
        "output_rails_duration": null,
        "total_duration": 0.01865863800048828,
        "llm_calls_duration": 0,
        "llm_calls_count": 0,
        "llm_calls_total_prompt_tokens": 0,
        "llm_calls_total_completion_tokens": 0,
        "llm_calls_total_tokens": 0
      }
    }
  }
}
```

```bash
curl -s -X POST "https://$NEMO_ROUTE/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "'"$LLM_MODEL_NAME"'",
    "messages": [{"role":"user","content":"My password for bot account is abc123456"}]
  }' | jq
```

should return something along the lines of

```json
{
  "id": "chatcmpl-bf0cd229-5896-44c3-95ea-ba5d608e750d",
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "I'm sorry, I can't respond to that.",
        "role": "assistant"
      }
    }
  ],
  "created": 1782396814,
  "model": "microsoft-phi-4",
  "object": "chat.completion",
  "guardrails": {
    "config_id": "lemonade-stand"
  }
}
```

