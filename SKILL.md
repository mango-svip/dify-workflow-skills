---
name: dify-workflow-dsl
description: Build and edit Dify workflow DSL files. Use when creating new Dify workflows from scratch, modifying existing workflows (adding/removing nodes, changing connections), validating workflow structure, or designing LLM-based automation flows with code execution, conditional logic, loops, and error handling.
---

# Dify Workflow DSL Builder

Build, edit, and validate Dify workflow DSL (Domain-Specific Language) files for creating AI-powered automation workflows.

## Core Capabilities

1. **Create new workflows** - Generate complete workflow YAML files from descriptions
2. **Edit existing workflows** - Add, modify, or remove nodes and connections
3. **Validate workflows** - Check DSL syntax and structure
4. **Design complex flows** - Build workflows with LLM nodes, code execution, branching, loops, and error handling

## Quick Reference

**Node Types**: See [references/node_types.md](references/node_types.md) for complete list including:
- Core: start, end, llm, code, if-else, loop, iteration
- Data: variable-aggregator, variable-assigner, template-transform
- External: http-request, tool, knowledge-retrieval
- Advanced: parameter-extractor, question-classifier, list-operator, answer

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
python scripts/generate_id.py 5  # Generate 5 unique IDs
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

Variables use the format: `{{#node_id.field_name#}}`

**Common fields by node type**:
- LLM: `text`, `usage`
- Code: `[output_name]`, `error_message`, `error_type`
- Start: `[variable_name]`
- If-else: `condition_result`
- Loop: `output`, `iteration`

**Example**:
```yaml
prompt_template:
  - role: user
    text: "Process this: {{#1732007415808.user_input#}}"
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

**fail-branch** (recommended for code nodes):
- Creates alternative execution path on error
- Provides `error_message` and `error_type` variables
- Requires variable-aggregator to merge with success path

**default-value**:
- Returns a default value on error
- Continues main execution path
- Simpler but less robust

## Validation Checklist

Before finalizing a workflow, verify:

- [ ] All node IDs are unique and properly formatted (numeric strings)
- [ ] workflow has exactly one start node and at least one end node
- [ ] All edges reference valid source and target node IDs
- [ ] Edge handles match node types (source, fail-branch, true, false)
- [ ] Variable references use correct syntax: `{{#node_id.field#}}`
- [ ] All referenced variables exist in upstream nodes
- [ ] Branching paths merge at variable-aggregator before end
- [ ] Node positions are set for visual layout
- [ ] Required fields present for each node type

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
