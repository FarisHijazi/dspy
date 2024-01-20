```python
%load_ext autoreload
%autoreload 2
```

<img src="../../../../docs/images/DSPy8.png" alt="DSPy Image" height="150"/>


## Guide: **DSPy Modules**

[<img align="center" src="https://colab.research.google.com/assets/colab-badge.svg" />](https://colab.research.google.com/github/stanfordnlp/dspy/blob/main/docs/guides/signatures.ipynb)

### Quick Recap

This guide assumes you followed the [intro tutorial](../../../../intro.ipynb) to build your first few DSPy programs.

Remember that **DSPy program** is just Python code that calls one or more **DSPy modules**, like `dspy.Predict` or `dspy.ChainOfThought`, to use LMs.

### 1) What is a DSPy Module?

A **DSPy module** is a building block for programs that use LMs.

#TODO link signatures docs
<!-- - Each built-in module abstracts a **prompting technique** (like chain of thought or ReAct). Crucially, they are generalized to handle any [DSPy Signature](). -->
- Each built-in module abstracts a **prompting technique** (like chain of thought or ReAct). Crucially, they are generalized to handle any DSPy Signature.

- A DSPy module has **learnable parameters** (i.e., the little pieces comprising the prompt and the LM weights) and can be invoked (called) to process inputs and return outputs.

- Multiple modules can be composed into bigger modules (programs). DSPy modules are inspired directly by NN modules in PyTorch, but applied to LM programs.

### 2) Why should I use a DSPy Module?

TODO. I typically take this as self-evident, but I'll spell it out here.


```python
# Install `dspy-ai` if needed. Then set up a default language model.
# TODO: Add a graceful line for OPENAI_API_KEY.

try: import dspy
except ImportError:
    %pip install dspy-ai
    import dspy

dspy.configure(lm=dspy.OpenAI(model='gpt-3.5-turbo-1106'))
```

### 3) What DSPy Modules are currently built-in?

1. **[`dspy.Predict`](../../../api/modules/Predict.md)**:

2. **[`dspy.ChainOfThought`](../../../api/modules/ChainOfThought.md)**: 

3. **[`dspy.ProgramOfThought`](../../../api/modules/ProgramOfThought.md)**:

4. **[`dspy.ReAct`](../../../api/modules/ReAct.md)**:

5. **[`dspy.MultiChainComparison`](../../../api/modules/MultiChainComparison.md)**:


We also have some function-style modules:

6. **`dspy.majority`**:

### 4) How do I use a built-in module, like `dspy.Predict` or `dspy.ChainOfThought`?

Let's start with the most fundamental one, `dspy.Predict`. Internally, all of the others are just built using it!

#TODO link signatures docs
<!-- We'll assume you are already at least a little familiar with [DSPy signatures](), which are declarative specs for defining the behavior of any module we use in DSPy. -->
We'll assume you are already at least a little familiar with DSPy signatures, which are declarative specs for defining the behavior of any module we use in DSPy.

To use a module, we first **declare** it by giving it a signature. Then we **call** the module with the input arguments, and extract the output fields!


```python
sentence = "it's a charming and often affecting journey."  # example from the SST-2 dataset.

# 1) Declare with a signature.
classify = dspy.Predict('sentence -> sentiment')

# 2) Call with input argument(s). 
response = classify(sentence=sentence)

# 3) Access the output.
print(response.sentiment)
```

    Positive
    

When we declare a module, we can pass configuration keys to it.

Below, we'll pass `n=5` to request five completions. We can also pass `temperature` or `max_len`, etc.

Let's use `dspy.ChainOfThought`. In many cases, simply swapping `dspy.ChainOfThought` in place of `dspy.Predict` improves quality.


```python
question = "What's something great about the ColBERT retrieval model?"

# 1) Declare with a signature, and pass some config.
classify = dspy.ChainOfThought('question -> answer', n=5)

# 2) Call with input argument.
response = classify(question=question)

# 3) Access the outputs.
response.completions.answer
```




    ['One great thing about the ColBERT retrieval model is its superior efficiency and effectiveness compared to other models.',
     'Its ability to efficiently retrieve relevant information from large document collections.',
     'One great thing about the ColBERT retrieval model is its superior performance compared to other models and its efficient use of pre-trained language models.',
     'One great thing about the ColBERT retrieval model is its superior efficiency and accuracy compared to other models.',
     'One great thing about the ColBERT retrieval model is its ability to incorporate user feedback and support complex queries.']



Let's dicuss the output object here.

The `dspy.ChainOfThought` module will generally inject a `rationale` before the output field(s) of your signature.

Let's inspect the (first) rationale and answer!


```python
print(f"Rationale: {response.rationale}")
print(f"Answer: {response.answer}")
```

    Rationale: produce the answer. We can consider the fact that ColBERT has shown to outperform other state-of-the-art retrieval models in terms of efficiency and effectiveness. It uses contextualized embeddings and performs document retrieval in a way that is both accurate and scalable.
    Answer: One great thing about the ColBERT retrieval model is its superior efficiency and effectiveness compared to other models.
    

This is accessible whether we request one or many completions.

We can also access the different completions as a list of `Prediction`s or as several lists, one for each field.


```python
response.completions[3].rationale == response.completions.rationale[3]
```




    True



### 5) How do I use more complex built-in modules?

The others are very similar, `dspy.ReAct` and `dspy.ProgramOfThought` etc. They mainly change the internal behavior with which your signature is implemented!

More examples soon!

### 6) How do I compose multiple modules into a bigger program?

DSPy is just Python code that uses modules in any control flow you like. (There's some magic internally at `compile` time to trace your LM calls.)

What this means is that, you can just call the modules freely. No weird abstractions for chaining calls.

This is basically PyTorch's design approach for define-by-run / dynamic computation graphs. Refer to the intro tutorials for examples.