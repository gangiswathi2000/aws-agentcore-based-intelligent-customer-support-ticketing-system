# Customer Support Chatbot with Amazon Bedrock AgentCore

This project demonstrates how a customer support chatbot can be built around
a well-designed system prompt rather than separate routing logic or custom
classification components. The prompt handles task routing, information
gathering, and grounding, while the agent harness manages the conversation
loop, session continuity, and tool execution.

## The problem

A support chatbot for an online shop needs to do three very different jobs
depending on what the customer says:

•⁠ ⁠A customer reports something broken and needs a ticket filed with engineering.
•⁠ ⁠A customer asks a routine question about orders, shipping, returns, or
payments that is already answered in the shop's FAQ.
•⁠ ⁠A customer asks something that falls outside both cases and needs to be
pointed toward a human support representative.

The interesting part is not only identifying which category a message fits
into, but handling each case correctly once it is recognized.

## Design approach

_Collecting ticket information across multiple turns._ A ticket cannot be
filed with partial information — it needs a description, reproduction steps,
and the customer's environment (browser, OS, or device). Customers rarely
provide all three in a single message, so the prompt is designed to conduct
a brief structured interview: ask for exactly one missing piece at a time,
never re-ask what is already stated, and delay the ticket-filing tool call
until all required information is present. The harness preserves conversation
state across turns, so the chatbot only checks what is missing rather than
re-deriving known facts.

_Restricting answers to known information._ Rather than allowing the model
to reason about shop policy, the prompt restricts responses to only what is
present in an embedded FAQ document and explicitly defines how to handle
out-of-scope questions — treat them as a handoff case rather than attempting
to answer.

_Disambiguating edge cases._ Some messages naturally fit multiple categories —
a late delivery could be a bug or a platform question, as could a checkout
error or declined payment. The prompt spells out how each should be
classified, since unclear boundaries typically lead to inconsistent routing.

## Architecture

•⁠ ⁠**Agent Harness:** Manages the conversation loop, maintains session state,
and executes tool calls.
•⁠ ⁠**Ticket Filing Tool:** Implemented as a Lambda function and exposed through
a gateway, keeping business logic separate from the conversation flow.
•⁠ ⁠**Persistent Storage:** Tickets are stored in DynamoDB.
•⁠ ⁠**System Prompt:** Contains all routing rules, information-collection logic,
FAQ grounding, and tie-breaking guidance. This design makes iteration
rapid: modify the prompt, rebuild, and test immediately.

## Evaluation

Manual testing is useful during development but does not scale. A test suite
was created with representative prompts paired against expected behavior,
covering all three message types plus edge cases. Each test case produces a
full transcript that is scored by an LLM-as-a-judge process.
