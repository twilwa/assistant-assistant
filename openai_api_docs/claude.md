Here is the documentation for the Assistants, Messages, Threads, and Runs beta endpoints, extracted verbatim from the scraped webpages:

# Assistants

Beta

Build assistants that can call models and use tools to perform tasks.

Get started with the Assistants API

## The assistant object

Beta

Represents an `assistant` that can call the model and use tools.

id

string

The identifier, which can be referenced in API endpoints.

object

string

The object type, which is always `assistant`.

created_at

integer

The Unix timestamp (in seconds) for when the assistant was created.

name

string or null

The name of the assistant. The maximum length is 256 characters.

description

string or null

The description of the assistant. The maximum length is 512 characters.

model

string

ID of the model to use. You can use the List models API to see all of your available models, or see our Model overview for descriptions of them.

instructions

string or null

The system instructions that the assistant uses. The maximum length is 32768 characters.

tools

array

A list of tool enabled on the assistant. There can be a maximum of 128 tools per assistant. Tools can be of types `code_interpreter`, `retrieval`, or `function`.

Show possible types

file_ids

array

A list of file IDs attached to this assistant. There can be a maximum of 20 files attached to the assistant. Files are ordered by their creation date in ascending order.

metadata

map

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

## Create assistant

Beta

post https://api.openai.com/v1/assistants

Create an assistant with a model and instructions.

### Request body

model

Required

ID of the model to use. You can use the List models API to see all of your available models, or see our Model overview for descriptions of them.

name

string or null

Optional

The name of the assistant. The maximum length is 256 characters.

description

string or null

Optional

The description of the assistant. The maximum length is 512 characters.

instructions

string or null

Optional

The system instructions that the assistant uses. The maximum length is 32768 characters.

tools

array

Optional

Defaults to []

A list of tool enabled on the assistant. There can be a maximum of 128 tools per assistant. Tools can be of types `code_interpreter`, `retrieval`, or `function`.

Show possible types

file_ids

array

Optional

Defaults to [] 

A list of file IDs attached to this assistant. There can be a maximum of 20 files attached to the assistant. Files are ordered by their creation date in ascending order.

metadata

map

Optional

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

### Returns

An assistant object.

# Threads

Beta

Create threads that assistants can interact with.

Related guide: Assistants

## The thread object

Beta

Represents a thread that contains messages.

id

string

The identifier, which can be referenced in API endpoints.

object

string

The object type, which is always `thread`.

created_at

integer

The Unix timestamp (in seconds) for when the thread was created.

metadata

map

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

## Create thread

Beta

post https://api.openai.com/v1/threads

Create a thread.

### Request body

messages

array

Optional

A list of messages to start the thread with.

Show properties

metadata

map

Optional

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

### Returns

A thread object.

# Messages

Beta

Create messages within threads

Related guide: Assistants

## The message object

Beta

Represents a message within a thread.

id

string

The identifier, which can be referenced in API endpoints.

object

string

The object type, which is always `thread.message`.

created_at

integer

The Unix timestamp (in seconds) for when the message was created.

thread_id

string

The thread ID that this message belongs to.

role

string

The entity that produced the message. One of `user` or `assistant`.

content

array

The content of the message in array of text and/or images.

Show possible types

assistant_id

string or null

If applicable, the ID of the assistant that authored this message.

run_id

string or null

If applicable, the ID of the run associated with the authoring of this message.

file_ids

array

A list of file IDs that the assistant should use. Useful for tools like retrieval and code_interpreter that can access files. A maximum of 10 files can be attached to a message.

metadata

map

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

## Create message

Beta

post https://api.openai.com/v1/threads/{thread_id}/messages

Create a message.

### Path parameters  

thread_id

string

Required

The ID of the thread to create a message for.

### Request body

role

string

Required

The role of the entity that is creating the message. Currently only `user` is supported.

content

string

Required

The content of the message.

file_ids

array

Optional

Defaults to []

A list of File IDs that the message should use. There can be a maximum of 10 files attached to a message. Useful for tools like `retrieval` and `code_interpreter` that can access and use files.

metadata

map

Optional

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

### Returns

A message object.

# Runs

Beta

Represents an execution run on a thread.

Related guide: Assistants  

## The run object

Beta

Represents an execution run on a thread.

id

string

The identifier, which can be referenced in API endpoints.

object

string  

The object type, which is always `assistant.run`.

created_at

integer

The Unix timestamp (in seconds) for when the run was created.

thread_id

string

The ID of the thread that was executed on as a part of this run.

assistant_id

string

The ID of the assistant used for execution of this run.

status

string

The status of the run, which can be either `queued`, `in_progress`, `requires_action`, `cancelling`, `cancelled`, `failed`, `completed`, or `expired`.

required_action

object or null

Details on the action required to continue the run. Will be `null` if no action is required. 

Show properties

last_error  

object or null

The last error associated with this run. Will be `null` if there are no errors.

Show properties

expires_at

integer  

The Unix timestamp (in seconds) for when the run will expire.

started_at

integer or null

The Unix timestamp (in seconds) for when the run was started.  

cancelled_at

integer or null

The Unix timestamp (in seconds) for when the run was cancelled.

failed_at

integer or null

The Unix timestamp (in seconds) for when the run failed.

completed_at  

integer or null

The Unix timestamp (in seconds) for when the run was completed.

model

string

The model that the assistant used for this run.

instructions

string

The instructions that the assistant used for this run.

tools  

array

The list of tools that the assistant used for this run.

Show possible types

file_ids

array

The list of File IDs the assistant used for this run.

metadata

map

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

## Create run

Beta

post https://api.openai.com/v1/threads/{thread_id}/runs

Create a run.

### Path parameters

thread_id  

string  

Required

The ID of the thread to run.

### Request body

assistant_id

string  

Required

The ID of the assistant to use to execute this run.

model

string or null

Optional

The ID of the Model to be used to execute this run. If a value is provided here, it will override the model associated with the assistant. If not, the model associated with the assistant will be used.

instructions

string or null

Optional  

Override the default system message of the assistant. This is useful for modifying the behavior on a per-run basis.

tools

array or null

Optional

Override the tools the assistant can use for this run. This is useful for modifying the behavior on a per-run basis.

Show possible types

metadata

map

Optional

Set of 16 key-value pairs that can be attached to an object. This can be useful for storing additional information about the object in a structured format. Keys can be a maximum of 64 characters long and values can be a maxium of 512 characters long.

### Returns

A run object.