# AGENT Instructions

## MCP Server Configuration

Always use the configured "omnistrate-ctl" MCP server when answering questions related to Omnistrate platform, services, or documentation.
When available use the --output json format to get the results of the command on json format.

## Answering questions about Omnistrate

### Query optimization

When processing Omnistrate-related questions:

1. **Search Strategy**: Use the omnistrate MCP server to search relevant documentation before providing answers
2. **Context Analysis**: Analyze the retrieved context thoroughly to understand the scope and specifics of the question
3. **Clarification**: Ask for clarification when the question is ambiguous or lacks sufficient detail to provide an accurate response
4. **Source-Based Responses**: Only provide answers that are directly supported by the context retrieved from the omnistrate MCP server

### Response Guidelines

- **Concrete Answers**: Provide specific, actionable responses based on the retrieved documentation
- **Context Dependency**: Do not answer Omnistrate-related questions without first consulting the MCP server
- **Accuracy**: If the MCP server doesn't provide sufficient context, explicitly state that more information is needed
- **No Speculation**: Avoid making assumptions or providing general knowledge about Omnistrate that isn't supported by the retrieved context

### Example Workflow

1. Receive Omnistrate-related question
2. Query omnistrate MCP server for relevant documentation
3. Analyze retrieved context
4. If context is sufficient: Provide concrete answer based on documentation
5. If context is insufficient: Ask for clarification or state that documentation doesn't cover the topic

## Generating and Updating Omnistrate compose specification

### Compose generation

When generating Omnistrate compose spec:

1. **File generation**: ALWAYS GENERATE THE COMPOSE SPEC IN A FILE NAMED: "compose.yaml"
2. **Identify the values needed**: Read the instructions in ./.omnistrate/COMPOSE-SPEC.md and ./.omnistrate/COMPOSE-EXTENSIONS.md to identify the requirements and features supported to specify a service
3. **Review the examples**: Read the examples in ./examples/features and ./examples/service folder to use as a reference
4. **Context Analysis**: Analyze the retrieved context thoroughly to understand the service specification to generate
5. **Clarification**: Ask for clarification when the question is ambiguous or lacks sufficient detail to provide an accurate response
6. **Validate**: ALWAYS VALIDATE THE GENERATED SPEC using "omnistrate-ctl" MCP server build command with dry-run option

### Generation guideline

- **Ask for clarification**: When presented with options on properties on the generated files, always ask for clarification instead of making things up
- **Iterative validation**:  If a generated spec does not confirm with the experience iterate to regenerate
- **No Speculation**: Avoid making assumptions or providing general knowledge about Omnistrate that isn't supported by the retrieved context

### Validation of generated compose specification

### Example Workflow to generate compose spec

1. Receive request to generate compose file question
2. Read spec file for COMPOSE and COMPOSE-EXTENSIONS and EXAMPLES
3. Analyze the request to understand the details of the service
4. If context is sufficient: Generate the spec file "compose.yaml"
5. If context is insufficient: Ask for clarification
6. Validate the spec file with a dry-run build
7. If the spec is valid: Complete the process
8. If the spec is invalid: Restart the process and regenerate the spec
