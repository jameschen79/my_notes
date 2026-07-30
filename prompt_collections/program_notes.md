# Open Source For Prompt Engineering

## LLMLingua

```python
from llmlingua import PromptCompressor

llm_lingua = PromptCompressor()
result = llm_lingua.compress_prompt(prompt, target_token=1024)
```

## Congee - memory for agent

```python
import congee

await congee.add("FILE_NAME")
await cognee.cognify()
await cognee.memify()
results = await cognee.search("QUESTION")
```

The results depends on the model.

## DSPy

```python
import dspy

class QA(dspy.Signature):
    question: str = dspy.InputField
    reasoning: str = dspy.OutputField
    answer: str = dspy.OutputField()

qa = dspy.ChainofThought(QA)
result = qa(question="QUESTION")
```
