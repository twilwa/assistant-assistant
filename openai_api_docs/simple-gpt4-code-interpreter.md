

# Build Custom Code Interpreter with E2B and GPT-4

This is an example of how to build a custom simple code interpreter that can
execute JavaScript using OpenAI GPT-4 and E2B.

##

What is a Code Interpreter?

Code Interpreter is a ChatGPT plugin released by OpenAI that gives ChatGPT
capabilities to run code.

This guide will show you how to build your own custom code interpreter that
can execute JavaScript code using E2B and GPT-4.

You can find the final code for both Python and JavaScript in this GitHub
repository.

## Why to use E2B?

E2B allows you to execute the GPT-generated code in a sandboxed cloud
environment. This means you can run the code without worrying about security
issues and the potential harm the generated code can cause to your machine.

## Install E2B and OpenAI

First, install the required packages. We need to install the E2B package and
OpenAI package.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    npm install @e2b/sdk
    npm install openai
    

CopyCopied!

## Get API keys

Let's get our API keys for E2B and OpenAI. We'll need them to use the APIs.

  1. Get your E2B API Key from here
  2. Get your GPT-4 or GPT-3.5 at the OpenAI website

## Import packages

Now we need to import the packages we installed in the previous step.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    import e2b from '@e2b/sdk'
    import OpenAI from 'openai'
    

CopyCopied!

## Make calls to GPT

Here we set up the OpenAI chat completion endpoint that allows to call the
GPT-4 (or GPT-3.5) models.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    const openai = new OpenAI()
    const chatCompletion = await openai.chat.completions.create({
      model: 'gpt-4', // Or use 'gpt-3.5-turbo'
      messages: [
        {
          role: 'system',
          content: 'You are a senior developer who can code in JavaScript. Always produce valid JSON.',
        }
      ],
    })
    

CopyCopied!

If you print the response, you should see something like this:

Run

    
    
    {
      "id": "chatcmpl-846e96G7wm4s7dSqgWvnaPaanyQ3j",
      "object": "chat.completion",
      "created": 1695989553,
      "model": "gpt-4-0613",
      "choices": [
        {
          "index": 0,
          "message": {
            "role": "assistant",
            "content": "Great! How can I assist you today? Are you looking for help with JavaScript solutions, or do you need some general advice about software development?"
          },
          "finish_reason": "stop"
        }
      ],
      "usage": {
        "prompt_tokens": 18,
        "completion_tokens": 29,
        "total_tokens": 47
      }
    }
    

CopyCopied!

## Prepare OpenAI functions

We're going to take advantage of the OpenAI functions. Defining these
functions will make it easier to instruct the model that it can write
JavaScript code to complete requests from the user. We'll execute this code
later using E2B.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    const functions = [
      {
        name: 'exec_code',
        description: 'Executes the passed JavaScript code using Nodejs and returns the stdout and stderr',
        parameters: {
          type: 'object',
          properties: {
            code: {
              type: 'string',
              description: 'The JavaScript code to execute.',
            },
          },
          required: ['code'],
        },
      },
    ]
    

CopyCopied!

We created an OpenAI function `exec_code` that expects a single parameter
`code`. The `code` parameter will be the Python code generated by GPT that
we'll execute.

Now we pass the `functions` variable to the GPT call we made earlier and also
add a few messages to show the model how to use the `exec_code` function. The
new code is marked by the highlighted lines.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    const chatCompletion = await openai.chat.completions.create({
      model: 'gpt-4',
      messages: [
        {
          role: 'system',
          content: 'You are a senior developer who can code in JavaScript. Always produce valid JSON.',
        },
        { 
          role: 'user', 
          content: 'Write hello world', 
        }, 
        { 
          role: 'assistant', 
          content: '{"code": "print("hello world")"}', 
          name: 'exec_code', 
        }, 
        { 
          role: 'user', 
          content: 'Generate first 100 fibonacci numbers', 
        }, 
      ],
      functions, 
    })
    

CopyCopied!

If you print the GPT response now, you'll most likely see the model is calling
the `exec_code` function we defined earlier and is passing code for generating
Fibonacci numbers in the first element of the `choices` JSON array.

Run

    
    
    {
      ...
      "choices": [
        {
          "index": 0,
          "message": {
            "role": "assistant",
            "content": null,
            "function_call": {
              "name": "exec_code",
              "arguments": "{\n  \"code\": \"\n    var fibonacciSequence = [0, 1];\n    for(var i=2; i<100; i++){\n        fibonacciSequence[i] = fibonacciSequence[i-1] + fibonacciSequence[i-2];\n    }\n    console.log(fibonacciSequence);\n  \"\n}"
            }
          },
          "finish_reason": "function_call"
        }
      ],
      ...
    }
    

CopyCopied!

## Parse GPT response

We need to parse the GPT's response and detect if it contains a function call.
If it does and it's actually the `exec_code` function, we need to parse the
`arguments` field, get the `code` argument and actually run the code using E2B
(we'll do it in the next step).

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    // ... previous code
    
    const message = chatCompletion.choices[0].message;
    const func = message["function_call"];
    if (func) {
      const funcName = func["name"];
    
      // Get rid of newlines and leading/trailing spaces in the raw function arguments JSON string.
      // This sometimes help to avoid JSON parsing errors.
      let args = func["arguments"];
      args = args.trim().replace(/\n|\r/g, "");
      // Parse the cleaned up JSON string.
      const funcArgs = JSON.parse(args);
    
      // If the model is calling the exec_code function we defined in the `functions` variable, we want to save the `code` argument to a variable.
      if (funcName === "exec_code") {
        const code = funcArgs["code"];
        // TODO: Execute the code using E2B.
      }
    } else {
      // The model didn't call a function, so we just printed the message.
      const content = message["content"];
      console.log(content);
    }
    

CopyCopied!

## Run GPT-generated code with E2B

It's time to actually run the code generated by GPT. We'll be using the
`e2b.runCode`/`e2b.run_code` to execute the code in E2B's sandboxed
playground. All we need to add is just a single line of code. I highlighted it
in the code snippets below.

![](/docs/_next/static/media/node.ffbff9e8.svg)JavaScript & TypeScript

![](/docs/_next/static/media/python.c624d255.svg)Python

    
    
    // ... previous code
    
    // If the model is calling the exec_code function we defined in the `functions` variable, we want to save the `code` argument to a variable.
    if (funcName === "exec_code") {
      const code = funcArgs["code"];
      // Execute the code using E2B.
      const { stdout, stderr } = await e2b.runCode("Node16", code); 
      console.log(stdout); 
      console.error(stderr); 
    }
    
    // ... rest of the code
    

CopyCopied!

And this is what the executed code for the "Generate first 100 Fibonacci
numbers" prompt prints:

Run

    
    
    [
                          0,                    1,                     1,
                          2,                    3,                     5,
                          8,                   13,                    21,
                         34,                   55,                    89,
                        144,                  233,                   377,
                        610,                  987,                  1597,
                       2584,                 4181,                  6765,
                      10946,                17711,                 28657,
                      46368,                75025,                121393,
                     196418,               317811,                514229,
                     832040,              1346269,               2178309,
                    3524578,              5702887,               9227465,
                   14930352,             24157817,              39088169,
                   63245986,            102334155,             165580141,
                  267914296,            433494437,             701408733,
                 1134903170,           1836311903,            2971215073,
                 4807526976,           7778742049,           12586269025,
                20365011074,          32951280099,           53316291173,
                86267571272,         139583862445,          225851433717,
               365435296162,         591286729879,          956722026041,
              1548008755920,        2504730781961,         4052739537881,
              6557470319842,       10610209857723,        17167680177565,
             27777890035288,       44945570212853,        72723460248141,
            117669030460994,      190392490709135,       308061521170129,
            498454011879264,      806515533049393,      1304969544928657,
           2111485077978050,     3416454622906707,      5527939700884757,
           8944394323791464,    14472334024676220,     23416728348467684,
          37889062373143900,    61305790721611580,     99194853094755490,
         160500643816367070,   259695496911122560,    420196140727489660,
         679891637638612200,  1100087778366101900,   1779979416004714000,
        2880067194370816000,  4660046610375530000,   7540113804746346000,
       12200160415121877000, 19740274219868226000,  31940434634990100000,
       51680708854858330000, 83621143489848430000, 135301852344706760000,
      218922995834555200000
    ]
    

CopyCopied!

## Final code

You can find the final code for both Python and JavaScript in this GitHub
repository.



