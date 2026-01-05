---
name: datasheet
description: Extract structured information from integrated circuit and component datasheets (PDF files or URLs) and generate consistent markdown summaries. Use when the user requests to extract, summarize, analyze, or document information from IC/component datasheets, or when they provide a datasheet and want structured documentation. Triggers on phrases like "extract this datasheet", "summarize this datasheet", "analyze [component name]", "document this IC", or when working with datasheets for hardware design.
---

# Datasheet

Extract comprehensive, structured information from IC and component datasheets using a specialized autonomous agent.

## Overview

This skill delegates datasheet extraction to a specialized autonomous agent (`datasheet-agent`) that:
- Processes PDFs (local files) or URLs
- Extracts 6 structured sections:
  1. General Information (family, name, manufacturer, variants, features)
  2. Pinout (detailed pin tables with 100% accuracy requirement)
  3. Usage Information (operating sequences, timing, functional modes)
  4. Electrical Characteristics (absolute max ratings, recommended conditions, DC/AC specs)
  5. Package Information (types, dimensions, thermal characteristics)
  6. Application Examples (reference circuits, cascading, layout recommendations)
- Generates markdown files in `datasheets/` directory
- Ensures 100% accuracy for critical information (pinouts, electrical specs)

**Context Efficiency**: The agent processes the datasheet in an isolated context. PDF content is loaded into the agent's context and discarded when the agent completes, keeping the main conversation clean and minimizing token usage.

## How to Use

When a user requests datasheet extraction:

1. **Identify the input**: User provides either:
   - Local PDF path: `/path/to/datasheet.pdf`
   - URL: `https://example.com/datasheet.pdf`

2. **Validate input**:
   - For local files: Verify the file exists and is accessible
   - For URLs: Confirm the URL is well-formed

3. **Delegate to agent**:
   - Use the Task tool to invoke the `datasheet-agent` subagent
   - Provide clear context about the datasheet source (path or URL)
   - Let the agent execute the full 6-step extraction workflow autonomously

   Example delegation:
   ```
   "I'm delegating this datasheet extraction to the datasheet-agent.
   Please process [URL/path], extract all 6 sections with 100% accuracy
   (especially pinouts and electrical specs), and save the markdown output
   to the datasheets/ directory."
   ```

4. **Present results**: When the agent completes, show the user:
   - File location (e.g., `datasheets/74HC595.md`)
   - Component name and key characteristics
   - Brief summary of what was extracted
   - Any issues or notes from extraction (missing sections, ambiguities resolved, etc.)

## Agent Responsibilities

The `datasheet-agent` autonomously handles:
- **Step 1**: Receive and validate input (PDF path or URL)
- **Step 2**: Read datasheet using Read tool (PDF) or WebFetch (URL)
- **Step 3**: Extract 6 structured sections with 100% accuracy
- **Step 4**: Generate markdown file with proper formatting and naming
- **Step 5**: Verify completeness and accuracy
- **Step 6**: Return summary with file location and key component info

For full workflow details, see `.claude/agents/datasheet-agent.md`

## Example Invocations

User might say:
- "Extract this datasheet: https://www.ti.com/lit/ds/symlink/sn74hc595.pdf"
- "Summarize the ATmega328P datasheet for me"
- "I need documentation for this IC: /path/to/lm358.pdf"
- "Help me understand the 74HC595"
- "/datasheet https://example.com/datasheet.pdf"

## Reference Materials

- **Example output format**: See `references/example_output.md` for a complete example of expected markdown output
- **Agent workflow details**: See `../../.claude/agents/datasheet-agent.md` for complete extraction workflow

## Benefits of Agent-Based Architecture

- **Context efficiency**: PDF content stays in agent's isolated context, not main conversation
- **Clean conversation**: Main conversation only sees high-level progress and final summary
- **Token savings**: ~9,000+ tokens saved in main conversation for typical datasheet (PDF content discarded when agent completes)
- **Consistent output**: Agent follows same workflow every time, ensuring reliable results
- **Easy maintenance**: Update agent definition without changing skill interface
