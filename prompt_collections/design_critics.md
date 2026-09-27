 I want you to improve this design. To figure out what to focus on, use a Fable 5 subagent as a design critic.

Follow this procedure at each iteration:

- Capture a screenshot of the current design
- Invoke the critic in a fresh context, with just the screenshot, not the code, implementation details, or earlier iterations/critiques
- Ask it to evaluate the aesthetic that the design is going for, imagine how a top design studio would execute this aesthetic, then outline the biggest gaps
- Lastly, it should provide a score out of 10 indicating how close the current design is to that studio-level quality bar

Provide this guidance to the critic in its prompt:

- It should think high-level about the overall structure and composition as well as look at the fine details
- It should watch out for patterns that feel overdone, excessive, or otherwise obviously AI-generated, and penalize them
- It should provide tight, specific feedback, not vague prose
- It should be bold and opinionated, not rely on what’s safe or easy

Your work is only complete when the critic independently deems it 9/10 or higher. Do not put that criterion in the critic prompt; keep it objective in its scoring. Use the same critic prompt each time.

Follow by image generation

``` text
 The design is pretty plain. Add more personality using image generation. Consider shaders or 3D effects in combination with images to create more interesting visuals.

 For image generation, use this OpenAI API key (only use it locally, do not store it in the code or product): sk-a1b2c3d4…

 Verify that your work looks right frame-by-frame in the browser.
```
