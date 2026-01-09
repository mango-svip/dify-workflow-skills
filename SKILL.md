---
name: dify-workflow-skills
description: Build and edit Dify workflow DSL files. Use when creating new Dify workflows from scratch, modifying existing workflows (adding/removing nodes, changing connections), validating workflow structure, or designing LLM-based automation flows with code execution, conditional logic, loops, and error handling.
---

# Dify Workflow DSL Builder

Build, edit, and validate Dify workflow DSL (Domain-Specific Language) files for creating AI-powered automation workflows.

## Architecture Overview

Dify workflows use a **queue-based, event-driven architecture** with:
- **GraphEngine**: Central orchestrator managing workflow execution
- **WorkerPool**: Thread pool for parallel node execution
- **VariablePool**: Centralized variable management across nodes
- **EdgeProcessor**: Handles conditional routing and branch selection
- **Graph Validator**: Ensures workflow integrity before execution

## Core Capabilities

1. **Create new workflows** - Generate complete workflow YAML files from descriptions
2. **Edit existing workflows** - Add, modify, or remove nodes and connections
3. **Validate workflows** - Check DSL syntax and structure
4. **Design complex flows** - Build workflows with LLM nodes, code execution, branching, loops, and error handling
5. **Error handling strategies** - Implement fail-branch, retry, or default-value error handling

## Quick Reference

**Node Types**: See [references/node_types.md](references/node_types.md) for complete list including:
- **Core Flow**: start, end, answer
- **LLM & AI**: llm, agent, parameter-extractor
- **Logic & Control**: if-else, question-classifier, loop, iteration
- **Data Processing**: code, template-transform, variable-aggregator, assigner, list-operator, document-extractor
- **External Integration**: http-request, tool, knowledge-retrieval, knowledge-index, datasource
- **Advanced**: trigger-webhook, trigger-schedule, trigger-plugin, human-input

**Workflow Structure**: See [references/workflow_structure.md](references/workflow_structure.md) for complete DSL format

**Templates**: Check [assets/](assets/) for example workflows

## Creating a New Workflow

### Step 1: Understand Requirements

Ask the user:
- What should the workflow do?
- What are the inputs and outputs?
- What processing steps are needed?
- Any conditional logic or error handling?

### Step 2: Design Node Flow

Plan the workflow structure:

1. **Start node** - Define input variables
2. **Processing nodes** - LLM, code, tools, etc.
3. **Control flow** - if-else for branching, loop for iteration
4. **Error handling** - Use fail-branch on code nodes
5. **Output aggregation** - variable-aggregator to merge branches
6. **End node** - Define outputs

### Step 3: Generate Node IDs

Use `scripts/generate_id.py` to create unique node IDs:

```bash
python3 scripts/generate_id.py 5  # Generate 5 unique IDs
```

Or generate in Python:
```python
import time
node_id = str(int(time.time() * 1000))
```

### Step 4: Build the Workflow

Create the complete YAML structure with:
- Top-level metadata (kind, version, app)
- App section (name, description, icon)
- Workflow section with features and graph
- Nodes array with positions
- Edges array connecting nodes

**Template to use**: Start from `assets/simple_llm_workflow.yml` for basic flows

### Step 5: Validate

Check the workflow structure for common issues:
- All nodes have unique IDs
- All edges reference existing node IDs
- Start and end nodes exist
- Required fields are present
- Variable references use correct syntax: `{{#node_id.field#}}`

## Common Workflow Patterns

### Pattern 1: Simple LLM Flow

Start → LLM → End

**Use case**: Basic question answering, text generation

**Template**: `assets/simple_llm_workflow.yml`

### Pattern 2: Error Handling Flow

Start → Code (with fail-branch) → [Success → Aggregator] / [Fail → LLM Recovery → Retry → Aggregator] → End

**Use case**: Robust data processing with error recovery

**Template**: `assets/error_handling_workflow.yml`

**Key points**:
- Set `error_strategy: fail-branch` on code node
- Fail branch provides `error_message` and `error_type` variables
- Use variable-aggregator to merge success/fail branches
- Reference aggregator output in end node

### Pattern 3: Conditional Routing

Start → If-Else → [True → Handler A] / [False → Handler B] → Aggregator → End

**Use case**: Route requests based on content/type

**Template**: `assets/conditional_workflow.yml`

**Key points**:
- If-else has `true` and `false` sourceHandles
- Use comparison operators: contains, starts_with, is_empty, etc.
- Combine conditions with logical_operator: "and" or "or"

### Pattern 4: Loop Processing

Start → Loop → [Process Items] → Loop End → End

**Use case**: Iterative processing with break conditions

**Key points**:
- Set `loop_count` for maximum iterations
- Define `break_conditions` to exit early
- Loop maintains state across iterations

## Editing Existing Workflows

### Adding a Node

1. Read the existing workflow file
2. Generate a new unique node ID
3. Add node to `workflow.graph.nodes` array with:
   - Unique ID and position
   - Node type and configuration
   - Proper sourcePosition/targetPosition
4. Add edge(s) to `workflow.graph.edges` connecting the new node
5. Update any dependent nodes (e.g., end node outputs)

### Removing a Node

1. Identify the node ID to remove
2. Remove node from `workflow.graph.nodes`
3. Remove all edges referencing that node ID (as source or target)
4. Update downstream nodes that referenced the removed node's outputs

### Modifying Connections

1. Find edge by source and target node IDs
2. Update edge `source`, `target`, `sourceHandle`, or `targetHandle`
3. Update edge `data.sourceType` and `data.targetType` to match actual node types
4. Update edge `id` to follow naming convention: `{source_id}-{handle}-{target_id}-target`

## Variable Reference Syntax

Variables are managed in a centralized **VariablePool** and use the format: `{{#node_id.field_name#}}`

### Variable Selector Pattern

The variable pattern regex: `{{#[a-zA-Z0-9_]{1,50}(?:\.[a-zA-Z_][a-zA-Z0-9_]{0,29}){1,10}#}}`

**Selector structure**: `[node_id, variable_name, ...optional_nested_keys]`
- First element: node ID that produced the variable
- Second element: variable name or output field
- Additional elements: nested object keys or array indices (for FileSegment/ObjectSegment)

### System Variables

Available via `sys` node ID:
- `{{#sys.query#}}` - User query/input
- `{{#sys.files#}}` - Uploaded files
- `{{#sys.conversation_id#}}` - Current conversation ID
- `{{#sys.user_id#}}` - User identifier
- `{{#sys.dialogue_count#}}` - Number of dialogue turns
- `{{#sys.app_id#}}` - Application ID
- `{{#sys.workflow_id#}}` - Workflow ID
- `{{#sys.workflow_run_id#}}` - Current execution ID
- `{{#sys.timestamp#}}` - Current timestamp

### Common Output Fields by Node Type

**LLM Node**:
- `{{#node_id.text#}}` - Generated text response
- `{{#node_id.usage#}}` - Token usage information
- `{{#node_id.reasoning_content#}}` - Model reasoning (if enabled)

**Code Node**:
- `{{#node_id.output_name#}}` - Named outputs defined in node config
- `{{#node_id.error_message#}}` - Error message (fail-branch only)
- `{{#node_id.error_type#}}` - Error type (fail-branch only)

**Start Node**:
- `{{#node_id.variable_name#}}` - Input variables defined in start node

**If-Else Node**:
- `{{#node_id.condition_result#}}` - Boolean condition result

**Loop Node**:
- `{{#node_id.output#}}` - Loop output array
- `{{#node_id.iteration#}}` - Current iteration number

**HTTP Request Node**:
- `{{#node_id.body#}}` - Response body
- `{{#node_id.status_code#}}` - HTTP status code
- `{{#node_id.headers#}}` - Response headers
- `{{#node_id.files#}}` - Downloaded files (if response is file)

**Knowledge Retrieval Node**:
- `{{#node_id.result#}}` - Retrieved knowledge segments

**Variable Aggregator**:
- `{{#node_id.output#}}` - Aggregated output from merged branches

### Environment and Conversation Variables

**Environment variables** (defined at app level):
- Access via special node ID: `env`
- Example: `{{#env.API_KEY#}}`

**Conversation variables** (session state):
- Access via special node ID: `conversation`
- Example: `{{#conversation.user_context#}}`

### Example Variable Usage

```yaml
# In LLM prompt template
prompt_template:
  - role: system
    text: "You are a helpful assistant."
  - role: user
    text: "Process this input: {{#1732007415808.user_input#}}"

# In code node
variables:
  - ["1732007415808", "user_input"]
  - ["1732007420123", "processed_data"]

# Accessing nested object fields
text: "File name: {{#upload_node.files.name#}}"
text: "API response status: {{#http_node.status_code#}}"
```

## Node Positioning

Position nodes on the canvas for visual clarity:

**Horizontal spacing**: 300-400 pixels between connected nodes
**Vertical spacing**:
- Same level: same y-coordinate
- Branches: offset by 150-200 pixels

**Example positions**:
```yaml
Start: x=100, y=250
LLM: x=400, y=250
Code: x=700, y=250
End: x=1000, y=250
```

For branching:
```yaml
If-else: x=400, y=300
True branch: x=700, y=200
False branch: x=700, y=400
```

## Error Handling Strategy

### Error Strategies

Dify supports multiple error handling strategies defined in the node configuration:

**1. fail-branch** (recommended for code/http nodes):
- Creates alternative execution path on error
- Provides `error_message` and `error_type` variables
- Uses `sourceHandle: "fail-branch"` in edge configuration
- Success path uses `sourceHandle: "success-branch"`
- Requires variable-aggregator to merge with success path before continuing to end node
- Allows graceful error recovery and custom error handling logic

**2. default-value**:
- Returns a predefined default value on error
- Continues main execution path without branching
- Simpler but less robust than fail-branch
- Good for non-critical operations where fallback values are acceptable

**3. abort** (default):
- Stops the entire workflow execution on failure
- No additional configuration needed
- Used when errors are unrecoverable

**4. retry**:
- Configurable retry logic with max retries and intervals
- Defined in node's `retry_config` section
- Useful for transient failures (network issues, rate limits)

### Implementing Fail-Branch Error Handling

In node configuration:
```yaml
error_strategy: fail-branch
```

In edges array:
```yaml
# Success path
- id: code_node-success-branch-next_node-target
  source: code_node_id
  target: aggregator_id
  sourceHandle: success-branch
  targetHandle: target

# Failure path
- id: code_node-fail-branch-error_handler-target
  source: code_node_id
  target: error_handler_id
  sourceHandle: fail-branch
  targetHandle: target
```

**Available error variables** in fail-branch:
- `{{#node_id.error_message#}}` - Human-readable error description
- `{{#node_id.error_type#}}` - Error type classification

## Validation Checklist

Before finalizing a workflow, verify:

### Structural Validation
- [ ] All node IDs are unique and properly formatted (numeric strings or valid identifiers)
- [ ] Workflow has exactly one root node (start, datasource, or trigger node)
- [ ] Workflow has at least one end node
- [ ] All edges reference valid source and target node IDs that exist in the nodes array
- [ ] No circular dependencies that would create infinite loops (except intentional loop nodes)
- [ ] Root node is correctly identified and accessible

### Edge and Connection Validation
- [ ] All edge IDs are unique
- [ ] Edge `sourceHandle` values match node types:
  - Standard nodes: `"source"` (default)
  - If-else/Question-classifier: `"true"`, `"false"`, or classification label
  - Code/HTTP with error handling: `"success-branch"` or `"fail-branch"`
  - Loop nodes: `"loop"` for continuation
- [ ] Edge `targetHandle` is typically `"target"` for most nodes
- [ ] Edge `data.sourceType` matches the actual source node type
- [ ] Edge `data.targetType` matches the actual target node type
- [ ] Branching paths (from if-else, question-classifier) have edges for all possible outcomes
- [ ] Fail-branch edges exist when `error_strategy: fail-branch` is used

### Variable and Data Flow Validation
- [ ] Variable references use correct syntax: `{{#node_id.field#}}`
- [ ] All referenced variables exist in upstream nodes (nodes that execute before current node)
- [ ] Variable selectors match actual output fields of referenced nodes
- [ ] System variables use correct `sys` prefix: `{{#sys.query#}}`
- [ ] Environment variables use `env` prefix if needed
- [ ] No undefined variable references that would cause runtime errors

### Error Handling Validation
- [ ] Nodes with `error_strategy: fail-branch` have both success and fail edges
- [ ] Branching paths merge at variable-aggregator before reaching end node
- [ ] Variable aggregator receives inputs from all branch paths
- [ ] Default values are provided when using `error_strategy: default-value`
- [ ] Retry configurations are valid (max retries, intervals) if using retry strategy

### Node Configuration Validation
- [ ] Required fields are present for each node type (see node_types.md)
- [ ] Node positions are set for visual layout (x, y coordinates)
- [ ] LLM nodes have valid model configurations (provider, name, mode)
- [ ] Code nodes specify valid language (python3 or javascript)
- [ ] HTTP request nodes have valid URLs and methods
- [ ] Loop nodes have valid `loop_count` and break conditions
- [ ] If-else nodes have properly structured conditions with valid operators

### Workflow Execution Validation
- [ ] No nodes are unreachable (all nodes can be reached from root node)
- [ ] All execution paths eventually lead to an end node or answer node
- [ ] Container nodes (iteration, loop) have proper start/end node pairs
- [ ] Human-input nodes are used appropriately (workflow will pause for input)
- [ ] Trigger nodes (webhook, schedule) are not mixed with standard start nodes

## Node Execution Types

Dify categorizes nodes by their execution behavior. Understanding these types helps design correct workflows:

### EXECUTABLE (Standard Logic Nodes)
Execute logic and produce outputs. Most common node type.
- **Examples**: llm, code, http-request, knowledge-retrieval, template-transform
- **Behavior**: Execute when all input dependencies are satisfied
- **Source Handle**: `"source"` (or `"success-branch"`/`"fail-branch"` with error handling)
- **Outputs**: Defined by node type (see Variable Reference section)

### BRANCH (Conditional Routing Nodes)
Control flow by choosing between multiple paths based on conditions.
- **Examples**: if-else, question-classifier
- **Behavior**: Evaluate conditions and activate one or more output edges
- **Source Handles**:
  - If-else: `"true"`, `"false"`
  - Question-classifier: classification labels (custom)
- **Important**: Unselected paths are marked as "skipped" and don't execute downstream

### CONTAINER (Sub-graph Management)
Manage nested execution contexts with iterations or loops.
- **Examples**: iteration, loop
- **Behavior**: Execute internal sub-graph multiple times
- **Components**:
  - Parent container node
  - Internal start node (iteration-start, loop-start)
  - Internal end node (iteration-end, loop-end)
- **Source Handle**: `"loop"` for iteration/loop continuation
- **State Management**: Maintain loop variables and iteration counts

### RESPONSE (Output Streaming)
Stream outputs to users in real-time.
- **Examples**: answer, end
- **Behavior**: Output results and potentially complete workflow
- **Usage**: Answer nodes can appear mid-workflow; End nodes terminate execution

### ROOT (Entry Points)
Serve as workflow entry points.
- **Examples**: start, datasource, trigger-webhook, trigger-schedule, trigger-plugin
- **Behavior**: First node to execute; no incoming edges
- **Important**: Only ONE root node per workflow
- **Constraint**: Standard start nodes and trigger nodes cannot coexist

## Workflow Execution Model

### Execution Flow

1. **Initialization**: GraphEngine enqueues root node into ReadyQueue
2. **Worker Execution**: Worker threads pull nodes from ReadyQueue and execute them
3. **Event Emission**: Workers push events (started, succeeded, failed) to event_queue
4. **Edge Processing**: Dispatcher processes events and identifies downstream nodes
5. **Dependency Resolution**: Downstream nodes added to ReadyQueue when dependencies satisfied
6. **Parallel Execution**: Multiple workers execute independent nodes concurrently
7. **Completion**: Workflow ends when End node executes or execution fails

### Edge States

Edges can be in three states during execution:
- **UNKNOWN**: Initial state, not yet evaluated
- **TAKEN**: Edge is traversed (condition met or path selected)
- **SKIPPED**: Edge is not traversed (condition failed or alternative path chosen)

### Workflow Execution Status

Workflows progress through these states:
- **SCHEDULED**: Queued but not yet started
- **RUNNING**: Currently executing
- **SUCCEEDED**: Completed without errors
- **FAILED**: Terminated due to unhandled error
- **PARTIAL_SUCCEEDED**: Completed with handled errors (via fail-branch or default-value)
- **STOPPED**: Manually stopped or aborted
- **PAUSED**: Waiting for human input (human-input node)

### Best Practices for Execution

1. **Parallel vs Sequential**: Independent nodes execute in parallel automatically. Use edges to enforce sequence when needed.
2. **Error Boundaries**: Use fail-branch on critical nodes to prevent workflow abortion
3. **Merge Points**: Always use variable-aggregator to merge branches before convergence
4. **Loop Safety**: Set reasonable `loop_count` limits and clear break conditions
5. **Human Input**: Plan for paused state when using human-input nodes

## Resources

### scripts/
- `generate_id.py` - Generate unique node IDs for workflows
- `validate_workflow.py` - Validate workflow DSL syntax (requires PyYAML)

### references/
- `node_types.md` - Complete reference of all Dify node types with examples
- `workflow_structure.md` - Detailed DSL structure and format specification

### assets/
- `simple_llm_workflow.yml` - Basic start→LLM→end template
- `error_handling_workflow.yml` - Template with fail-branch error handling
- `conditional_workflow.yml` - Template with if-else branching

## Tips for Success

1. **Start simple**: Begin with basic flows, add complexity incrementally
2. **Use templates**: Adapt existing templates rather than starting from scratch
3. **Validate early**: Check structure before adding many nodes
4. **Plan error handling**: Consider what can fail and how to handle it
5. **Name clearly**: Use descriptive node titles for maintainability
6. **Reference docs**: Consult node_types.md for field requirements
7. **Test incrementally**: Build and test workflows step by step
