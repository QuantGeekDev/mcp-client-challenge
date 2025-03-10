### MCP Client Challenge
By Alex Andru

This repo contains a very basic LLM chat streaming application that uses the OpenAI Client.
The challenge is to add a fully fledged MCP Client implementation to this.

I strongly suggest you do not use any AI help in order to fully enjoy this challenge and get some domain knowledge in MCPs - otherwise, why are you doing this challenge to begin with?


### Quickstart
Create a .env file based on the .env.example, and add your api key

Install packages with `npm i`
You can run the client with `npm run dev`


### Challenge steps

We will be using SSE transport. This means the MCP Server we will be connecting to will be available at a URL. This can be an [SSE mcp server on localhost](https://mcp-framework.com/docs/Transports/sse)

For testing, you can use this publicly available SSE server: https://calculator.mcp-framework.com/sse
1. Add a way for the user to write the URL for the server to connect to. This can be right on top of the chat, it doesn't have to be pretty.

2. [Figure out how to use that url to connect the client to the server]("https://github.com/modelcontextprotocol/typescript-sdk")
3. Add a "status" to your chat UI that shows if the server is connected, or disconnected. Set this status based on the results of a ping health check
4. When a server is connected successfully, it gets added to a list of currently connected servers that is displayed in the UI. when the connection attempt fails, the server's name doesn't get added to the list

5. Each server should display the number of tools it has, and can have a dropdown button that actually shows what these tools are


6. Each tool in the dropdown should have an input for the user to test the tool and receive the results in the browser. (to achieve this, you will need to find a way to call the tools in the remote mcp server - dig around in the mcpClient to figure out how)

7. Finally, you can connect the MCP server to the actual LLM... but how?
Let's take a look at OpenAI Client we have been calling to stream the chat completions. There seems to be an optional tools parameter we can pass. How does this work? We can list the tools in our MCP server. And if we look at our mcp tools, and we look at the tools expected by the openai client... something doesn't match. We can't just pass our mcp tools into the client - it gives us a type error. Something has to happen to our mcp tools so that openai's client can understand them. You need to figure out what and apply it. When you do, the llm will answer differently if you actually trigger a tool call with your prompt. You'll notice it doesn't write anything. That's how you know this part worked.

8. Now we can see that the LLM actually understands when we are trying to call a tool - if we are using the calculator tool, I can now ask it to sum "5 and 2" and it will glitch out in the chat, but asking other questions doesn't glitch out the chat. The LLM is actually responding with a content type that is different - it switches to a "tool_call" type of answer. Since we're streaming, this arrives in chunks. You need to write an accumulator of some kind to store the object that arives as a response from the openai client. It should have the name of a function, and a arguments property. So, if we ask our chat to sum 5 and 2, it might return something like (double check, i'm writing this challenge from loose memory) {name: "sum_tool", arguments: {a: "5", b:"2"}}

9. When the llm stops sending the stream for a tool call, it sends a 'finish reason' signal to indicate that it's done with the tool definition. [read more about that in the openai docs](https://platform.openai.com/docs/api-reference/introduction). You need to be able to detect when the response includes the finish reason. Once that happens, you need to turn our accumulated object / string from previous step (you can go about this in many ways) into an actual mcp client toolCall(), where you pass the tool name and arguments into it. This will actually send a function call request over JRPC to the mcp server. You need to capture the response to this function call.

10. You're almost done! You now need to take that tool call response, and start a new streaming generation request where you concatenate the tool call response at the end of the chat history, and ask the llm to continue. This allows the LLM to continue generating, with the new tool call result context in mind. And that's pretty much it for a basic implementation.

