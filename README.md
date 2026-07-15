# exe2c
Agent harness for smooth transition of any executable to a single .c file

## IDEA

I think about end to end transition of any executable into a simple portable programming language like c. 

As first example it would be interesting to take a simple format and platform.

Although modern frontier LLMs are capable to understand machine code and provide reasonable description, they still struggle with global context and especially important to produce 1:1 functionally the same but written in higher order programming language program description, directly portable and build-ready, but also human readable (although I am not sure about last point if is it strictly needed - coding agents don't care and many people today are using coding assistants already skipping reading any programming code). Thus, proposed system will:

- Take input executable file
- Properly translate pointers and perform block relocations
- Disassemble it into platform specific and decompile-ready form
- Build supporting representations (e.g. interm. code and data graphs)
- Decompile functions into C equivalents
- Split into distinct blocks capable for AI analysis
- Prepare supporing AGENTS.md and other memories for LLM coding agents
- Trigger LLM coding agents to write high quality portable C
