# Dify Workflow Node Types Reference

This document describes all available node types in Dify workflow DSL.

## Core Node Types

### 1. start
**Purpose**: Entry point of the workflow, defines input variables

**Required fields**:
- `type`: "start"
- `title`: Display name
- `variables`: Array of input variable definitions

**Variable fields**:
- `variable`: Variable name
- `label`: Display label
- `type`: Variable type (text, paragraph, number, select, etc.)
- `required`: Boolean
- `max_length`: Maximum length (for text types)
- `options`: Array of options (for select type)

**Example**:
```yaml
type: start
title: Start
variables:
  - label: User Input
    max_length: 10000
    required: true
    type: paragraph
    variable: user_input
```

### 2. end
**Purpose**: Terminal node that defines workflow outputs

**Required fields**:
- `type`: "end"
- `title`: Display name
- `outputs`: Array of output variable selectors

**Output fields**:
- `variable`: Output variable name
- `value_selector`: Array path to source value (e.g., ['node_id', 'field_name'])

**Example**:
```yaml
type: end
title: End
outputs:
  - variable: result
    value_selector:
      - '1733478262179'
      - text
  - variable: error_message
    value_selector:
      - '1733478343153'
      - error_message
```

### 3. llm
**Purpose**: Large Language Model inference node

**Required fields**:
- `type`: "llm"
- `title`: Display name
- `model`: Model configuration
- `prompt_template`: Array of message templates

**Model fields**:
- `provider`: Model provider (anthropic, openai, etc.)
- `name`: Model name (claude-3-5-sonnet-20241022, gpt-4, etc.)
- `mode`: chat or completion
- `completion_params`: Parameters like temperature

**Prompt template fields**:
- `id`: Unique identifier
- `role`: system, user, or assistant
- `text`: Template text with variable references {{#node_id.field#}}

**Context fields** (optional):
- `enabled`: Boolean
- `variable_selector`: Array of context variable paths

**Vision fields** (optional):
- `enabled`: Boolean

**Example**:
```yaml
type: llm
title: Generate Response
model:
  provider: anthropic
  name: claude-3-5-sonnet-20241022
  mode: chat
  completion_params:
    temperature: 0.7
prompt_template:
  - id: unique-id-1
    role: system
    text: You are a helpful assistant.
  - id: unique-id-2
    role: user
    text: "{{#1732007415808.user_input#}}"
context:
  enabled: false
  variable_selector: []
vision:
  enabled: false
```

### 4. code
**Purpose**: Execute Python or JavaScript code

**Required fields**:
- `type`: "code"
- `title`: Display name
- `code`: Code string
- `code_language`: python3 or javascript
- `variables`: Input variables
- `outputs`: Output type definitions

**Variable fields**:
- `variable`: Parameter name in code
- `value_selector`: Array path to input value

**Output fields**:
- `[output_name]`: Output variable name
  - `type`: Data type (string, number, object, array)
  - `children`: Nested structure (for objects)

**Error handling fields** (optional):
- `error_strategy`: "fail-branch" or "default-value"

**Example**:
```yaml
type: code
title: Process Data
code: |
  def main(input_text: str) -> dict:
      result = input_text.upper()
      return {'output': result}
code_language: python3
variables:
  - variable: input_text
    value_selector:
      - '1733478262179'
      - text
outputs:
  output:
    type: string
error_strategy: fail-branch
```

### 5. variable-aggregator
**Purpose**: Merge multiple conditional branch outputs into a single variable

**Required fields**:
- `type`: "variable-aggregator"
- `title`: Display name
- `variables`: Array of variable selector paths to aggregate
- `output_type`: Data type (string, number, object, array)

**Usage**: Required when downstream nodes need to reference values from multiple possible upstream branches (e.g., success/fail branches)

**Example**:
```yaml
type: variable-aggregator
title: Merge Results
output_type: object
variables:
  - - '1733478343153'
    - result
  - - '17334785192390'
    - result
```

### 6. if-else
**Purpose**: Conditional branching based on conditions

**Required fields**:
- `type`: "if-else"
- `title`: Display name
- `conditions`: Array of condition definitions
- `logical_operator`: "and" or "or"

**Condition fields**:
- `variable_selector`: Array path to value
- `comparison_operator`: is, is_not, contains, not_contains, starts_with, ends_with, is_empty, is_not_empty, greater_than, less_than, etc.
- `value`: Comparison value

**Example**:
```yaml
type: if-else
title: Check Condition
logical_operator: and
conditions:
  - variable_selector:
      - '1733478262179'
      - text
    comparison_operator: contains
    value: "error"
```

### 7. iteration
**Purpose**: Loop over an array/list

**Required fields**:
- `type`: "iteration"
- `title`: Display name
- `input_selector`: Array path to iterable
- `output_selector`: Array path to aggregated output

**Example**:
```yaml
type: iteration
title: Process Items
input_selector:
  - '1733478262179'
  - items
```

### 8. knowledge-retrieval
**Purpose**: Retrieve information from knowledge base

**Required fields**:
- `type`: "knowledge-retrieval"
- `title`: Display name
- `dataset_ids`: Array of dataset IDs
- `query_variable_selector`: Array path to query text

**Example**:
```yaml
type: knowledge-retrieval
title: Search Knowledge
dataset_ids:
  - dataset-id-1
query_variable_selector:
  - '1732007415808'
  - user_query
```

### 9. http-request
**Purpose**: Make HTTP API calls

**Required fields**:
- `type`: "http-request"
- `title`: Display name
- `method`: GET, POST, PUT, DELETE, PATCH
- `url`: Request URL
- `headers`: Request headers
- `body`: Request body (for POST/PUT/PATCH)

**Example**:
```yaml
type: http-request
title: API Call
method: POST
url: "https://api.example.com/endpoint"
headers:
  Content-Type: application/json
body:
  key: "{{#1732007415808.user_input#}}"
```

### 10. template-transform
**Purpose**: Transform variables using Jinja2 templates

**Required fields**:
- `type`: "template-transform"
- `title`: Display name
- `template`: Jinja2 template string
- `variables`: Input variables

**Example**:
```yaml
type: template-transform
title: Format Output
template: "Hello {{name}}, your score is {{score}}"
variables:
  - variable: name
    value_selector:
      - '1732007415808'
      - user_name
  - variable: score
    value_selector:
      - '1733478343153'
      - result
```

## Additional Node Types

### 11. answer
**Purpose**: Output answer to user in chatflow/workflow

**Required fields**:
- `type`: "answer"
- `title`: Display name
- `answer`: Answer template string with variable references

**Example**:
```yaml
type: answer
title: Answer
answer: "The result is: {{#1733478262179.text#}}"
```

### 12. loop
**Purpose**: Execute a set of nodes multiple times with break conditions

**Required fields**:
- `type`: "loop"
- `title`: Display name
- `loop_count`: Maximum number of iterations
- `break_conditions`: Conditions to break the loop
- `logical_operator`: "and" or "or"

**Example**:
```yaml
type: loop
title: Loop Processing
loop_count: 10
logical_operator: or
break_conditions:
  - variable_selector:
      - '1733478262179'
      - is_complete
    comparison_operator: is
    value: true
```

### 13. tool
**Purpose**: Call external tools or plugins

**Required fields**:
- `type`: "tool"
- `title`: Display name
- `provider_id`: Tool provider identifier
- `provider_type`: Tool provider type
- `tool_name`: Name of the tool
- `tool_parameters`: Tool-specific parameters

**Example**:
```yaml
type: tool
title: Call API Tool
provider_id: provider-123
provider_type: api
tool_name: fetch_data
tool_parameters:
  endpoint: "{{#1732007415808.api_endpoint#}}"
```

### 14. parameter-extractor
**Purpose**: Extract structured parameters from LLM output

**Required fields**:
- `type`: "parameter-extractor"
- `title`: Display name
- `model`: Model configuration
- `query`: Input text to extract from
- `parameters`: Parameter schema definitions

**Example**:
```yaml
type: parameter-extractor
title: Extract Parameters
model:
  provider: anthropic
  name: claude-3-5-sonnet-20241022
query:
  variable_selector:
    - '1732007415808'
    - user_input
parameters:
  - name: date
    type: string
    description: Extract the date mentioned
    required: true
```

### 15. variable-assigner
**Purpose**: Assign or transform variables

**Required fields**:
- `type`: "variable-assigner"
- `title`: Display name
- `variables`: Variable assignments

**Example**:
```yaml
type: variable-assigner
title: Assign Variables
variables:
  - variable: output_var
    value_selector:
      - '1733478262179'
      - text
```

### 16. question-classifier
**Purpose**: Classify user questions into categories

**Required fields**:
- `type`: "question-classifier"
- `title`: Display name
- `model`: Model configuration
- `query_variable_selector`: Input query
- `classes`: Classification categories

**Example**:
```yaml
type: question-classifier
title: Classify Question
model:
  provider: anthropic
  name: claude-3-5-sonnet-20241022
query_variable_selector:
  - '1732007415808'
  - user_query
classes:
  - name: technical
    description: Technical questions
  - name: general
    description: General questions
```

### 17. document-extractor
**Purpose**: Extract content from documents

**Required fields**:
- `type`: "document-extractor"
- `title`: Display name
- `variable_selector`: Document input

**Example**:
```yaml
type: document-extractor
title: Extract Document
variable_selector:
  - '1732007415808'
  - document_file
```

### 18. list-operator
**Purpose**: Perform operations on lists (filter, map, reduce)

**Required fields**:
- `type`: "list-operator"
- `title`: Display name
- `operation`: Operation type (filter, map, reduce, etc.)
- `variable_selector`: Input list

**Example**:
```yaml
type: list-operator
title: Filter List
operation: filter
variable_selector:
  - '1733478262179'
  - items
filter_condition:
  comparison_operator: greater_than
  value: 10
```

## Variable References

Variables are referenced using the syntax: `{{#node_id.field_name#}}`

**Examples**:
- `{{#1732007415808.user_input#}}` - Reference user_input from start node
- `{{#1733478262179.text#}}` - Reference text output from LLM node
- `{{#1733478343153.result#}}` - Reference result from code node
- `{{#1733478343153.error_message#}}` - Reference error_message from failed code node
- `{{#1733478343153.error_type#}}` - Reference error_type from failed code node

## Error Handling

### Fail Branch
Code nodes support `error_strategy: "fail-branch"` which creates an alternative execution path when the code fails.

**Handle**: `fail-branch` (vs. normal `source` handle)

**Available error variables**:
- `error_message`: Error description
- `error_type`: Error type/class

**Example flow**:
```
Code Node (with fail-branch)
  |-- source (success) --> Next Node
  |-- fail-branch (error) --> Error Handler Node
```

Both branches typically merge into a variable-aggregator before the end node.
