# Workflow Design Breakdown

## Overview

Detailed workflow design for the LLM Recommendation Agent, covering the complete process from user input to report generation.

## High-Level Workflow

```
User Input → Input Processing → Criteria Assembly → Data Collection → Evaluation → Report Generation → Output
```

---

## Phase 1: Input Processing

### 1.1 Receive User Input

**Input Types**:
- Natural language description of use case
- Screenshots/images (optional)
- Preference indicators
- Hardware constraints (for local deployment)
- Budget constraints

**Input Validation**:
- Check for minimum required information
- Validate image formats
- Parse structured preferences (if provided)
- Identify missing information

**Example Input**:
```
Text: "I need a model for system administration tasks - file management, bash commands, basic debugging.
       I want something cost-effective for daily use, maybe 80% as capable as the best models.
       I'll keep Claude Opus for complex projects."
Image: [Screenshot of SWE-Bench leaderboard]
```

### 1.2 Parse Natural Language

**Tasks**:
- Extract use case description
- Identify primary and secondary use cases
- Parse preference statements
- Extract constraints and requirements
- Identify comparative statements

**Output Structure**:
```json
{
  "useCase": {
    "primary": "system administration",
    "secondary": ["file management", "bash commands", "debugging"],
    "description": "daily use system administration tasks"
  },
  "preferences": {
    "costEffective": true,
    "capabilityTarget": 0.8,
    "referenceModel": "claude-opus"
  },
  "constraints": {
    "excludeModels": ["claude-opus"],
    "dailyUsage": true
  }
}
```

### 1.3 Analyze Images (if provided)

**Tasks**:
- Extract text from screenshots (OCR)
- Parse tabular data
- Identify model names and scores
- Extract benchmark results
- Recognize annotations and highlights

**Output Structure**:
```json
{
  "imageData": {
    "source": "swe-bench-leaderboard",
    "extractedModels": [
      {"name": "claude-3.5-sonnet", "score": 49.0, "cost": 0.45},
      {"name": "gpt-4o", "score": 47.2, "cost": 0.52}
    ],
    "confidence": 0.95
  }
}
```

### 1.4 Normalize and Combine Inputs

**Tasks**:
- Merge text and image data
- Resolve conflicts between sources
- Fill in missing information with defaults
- Validate combined input
- Generate summary for user confirmation

---

## Phase 2: Criteria Assembly

### 2.1 Generate Exclusion Criteria (Must-Haves)

**Process**:
- Analyze use case requirements
- Identify non-negotiable capabilities
- Apply domain knowledge
- Generate explicit exclusion filters

**Example Exclusion Criteria**:
```json
{
  "exclusionCriteria": [
    {
      "type": "capability",
      "name": "agentic_tool_calling",
      "required": true,
      "reason": "Required for system administration tasks"
    },
    {
      "type": "model",
      "name": "exclude_specific_models",
      "values": ["claude-opus"],
      "reason": "User will keep for complex projects"
    }
  ]
}
```

### 2.2 Generate Inclusion Criteria (Nice-to-Haves)

**Process**:
- Identify desirable features
- Assign relative importance weights
- Create scoring rubric
- Define measurement methods

**Example Inclusion Criteria**:
```json
{
  "inclusionCriteria": [
    {
      "name": "cost_effectiveness",
      "weight": 0.4,
      "measurement": "cost_per_task",
      "target": "low",
      "reason": "User wants cost-effective daily use"
    },
    {
      "name": "performance",
      "weight": 0.3,
      "measurement": "benchmark_score",
      "target": ">= 80% of SOTA",
      "reason": "80% as capable as best models"
    },
    {
      "name": "bash_capability",
      "weight": 0.2,
      "measurement": "community_feedback",
      "target": "positive",
      "reason": "System administration focus"
    },
    {
      "name": "context_window",
      "weight": 0.1,
      "measurement": "tokens",
      "target": ">= 100K",
      "reason": "Sufficient for most tasks"
    }
  ]
}
```

### 2.3 Validate Criteria

**Tasks**:
- Check for conflicting criteria
- Validate weight distribution (sum to 1.0)
- Ensure criteria are measurable
- Verify data source availability
- Generate criteria summary

---

## Phase 3: Data Collection

### 3.1 Query Hugging Face MCP

**Purpose**: Get model metadata and technical specifications

**Query Parameters**:
- Model names (from user input or image extraction)
- Filters for capabilities
- Tags for use cases

**Expected Data**:
```json
{
  "models": [
    {
      "id": "meta-llama/Llama-3.1-70B-Instruct",
      "name": "Llama 3.1 70B",
      "parameters": 70000000000,
      "contextWindow": 131072,
      "capabilities": ["text", "tool_calling"],
      "quantizations": ["4-bit", "8-bit", "16-bit"],
      "tags": ["instruct", "chat"]
    }
  ]
}
```

### 3.2 Query models.dev API

**Purpose**: Get benchmark scores, pricing, and standardized metrics

**Query Parameters**:
- Model IDs or names
- Benchmark filters (SWE-Bench, etc.)
- Provider filters

**Expected Data**:
```json
{
  "models": [
    {
      "id": "claude-3.5-sonnet",
      "benchmarks": {
        "swe-bench": {"score": 49.0, "rank": 1},
        "mmlu": {"score": 88.7, "rank": 3}
      },
      "pricing": {
        "input": "$3.00/1M tokens",
        "output": "$15.00/1M tokens"
      },
      "capabilities": {
        "toolCalling": true,
        "vision": true
      }
    }
  ]
}
```

### 3.3 Query Search MCP

**Purpose**: Get community feedback and real-world experiences

**Search Queries**:
- "{model_name} system administration"
- "{model_name} bash commands"
- "{model_name} vs {other_model}"
- "best LLM for sysadmin"

**Expected Data**:
```json
{
  "results": [
    {
      "source": "reddit",
      "url": "https://reddit.com/r/LocalLLaMA/comments/...",
      "title": "Claude 3.5 Sonnet for system administration",
      "sentiment": "positive",
      "keyPoints": [
        "Excellent at bash commands",
        "Good at debugging",
        "Reasonable cost"
      ],
      "relevanceScore": 0.9
    }
  ]
}
```

### 3.4 Aggregate and Normalize Data

**Tasks**:
- Merge data from all sources
- Normalize different data formats
- Handle missing data points
- Calculate derived metrics
- Flag inconsistencies

**Normalized Model Data**:
```json
{
  "modelId": "claude-3.5-sonnet",
  "name": "Claude 3.5 Sonnet",
  "provider": "Anthropic",
  "metadata": {
    "parameters": null,
    "contextWindow": 200000,
    "releaseDate": "2024-06-20",
    "knowledgeCutoff": "2024-04"
  },
  "benchmarks": {
    "sweBench": {"score": 49.0, "rank": 1},
    "mmlu": {"score": 88.7, "rank": 3}
  },
  "pricing": {
    "inputCostPerMillion": 3.0,
    "outputCostPerMillion": 15.0,
    "estimatedMonthlyCost": 45.0
  },
  "capabilities": {
    "toolCalling": true,
    "vision": true,
    "search": false
  },
  "community": {
    "sentimentScore": 0.85,
    "useCaseFeedback": {
      "systemAdministration": 0.9,
      "coding": 0.88,
      "debugging": 0.87
    }
  },
  "sources": ["huggingface", "models.dev", "search"]
}
```

---

## Phase 4: Evaluation

### 4.1 Apply Exclusion Filters

**Process**:
- Filter out models without required capabilities
- Remove excluded models
- Apply hard constraints
- Generate filtered candidate list

**Example**:
```json
{
  "candidates": [
    "claude-3.5-sonnet",
    "gpt-4o",
    "llama-3.1-70b"
  ],
  "excluded": [
    {"model": "claude-opus", "reason": "User excluded"},
    {"model": "gpt-3.5-turbo", "reason": "No tool calling"}
  ]
}
```

### 4.2 Score Models on Inclusion Criteria

**Process**:
- Apply each inclusion criterion
- Calculate scores using defined measurements
- Apply weights to each criterion
- Sum weighted scores for total

**Scoring Example**:
```json
{
  "model": "claude-3.5-sonnet",
  "scores": {
    "cost_effectiveness": {
      "rawScore": 7.5,
      "normalizedScore": 0.75,
      "weight": 0.4,
      "weightedScore": 0.30
    },
    "performance": {
      "rawScore": 49.0,
      "normalizedScore": 1.0,
      "weight": 0.3,
      "weightedScore": 0.30
    },
    "bash_capability": {
      "rawScore": 0.9,
      "normalizedScore": 0.9,
      "weight": 0.2,
      "weightedScore": 0.18
    },
    "context_window": {
      "rawScore": 200000,
      "normalizedScore": 1.0,
      "weight": 0.1,
      "weightedScore": 0.10
    }
  },
  "totalScore": 0.88
}
```

### 4.3 Generate Rankings

**Process**:
- Sort models by total score
- Apply tie-breaking rules if needed
- Generate confidence scores
- Identify top recommendations

**Rankings**:
```json
{
  "rankings": [
    {
      "rank": 1,
      "model": "claude-3.5-sonnet",
      "totalScore": 0.88,
      "confidence": 0.92
    },
    {
      "rank": 2,
      "model": "gpt-4o",
      "totalScore": 0.82,
      "confidence": 0.88
    },
    {
      "rank": 3,
      "model": "llama-3.1-70b",
      "totalScore": 0.75,
      "confidence": 0.80
    }
  ]
}
```

### 4.4 Cost-Effectiveness Analysis

**Process**:
- Calculate cost per task estimates
- Compare performance vs. cost ratios
- Identify best value options
- Generate cost projections

**Cost Analysis**:
```json
{
  "costEffectiveness": [
    {
      "model": "claude-3.5-sonnet",
      "costPerTask": 0.15,
      "performanceScore": 0.88,
      "valueRatio": 5.87,
      "monthlyEstimate": 45.0
    }
  ]
}
```

---

## Phase 5: Report Generation

### 5.1 Compile Executive Summary

**Content**:
- Top 3 recommendations
- Key findings
- Methodology overview
- Confidence assessment

### 5.2 Document Methodology

**Content**:
- User requirements analysis
- Exclusion criteria applied
- Inclusion criteria and weights
- Data sources used
- Scoring methodology

### 5.3 Generate Detailed Model Analysis

**Content per Model**:
- Overview and capabilities
- Benchmark performance
- Cost analysis
- Community feedback
- Suitability assessment
- Pros and cons

### 5.4 Include Data Lineage

**Content**:
- All data sources used
- Timestamps of data retrieval
- API calls made
- Confidence scores per data point
- Known limitations

### 5.5 Format Report

**Structure**:
```markdown
# LLM Recommendation Report

## Executive Summary
[Top recommendations and key findings]

## User Requirements Analysis
[Parsed requirements and constraints]

## Evaluation Methodology
[Criteria, weights, and scoring approach]

## Data Sources
[All sources with timestamps]

## Ranked Recommendations
### 1. [Model Name]
[Detailed analysis]

### 2. [Model Name]
[Detailed analysis]

## Cost Analysis
[Cost projections and comparisons]

## Data Lineage
[Source tracking and confidence scores]

## Appendix
[Raw data, calculations, references]
```

---

## Phase 6: Output and Delivery

### 6.1 Save Report

**Actions**:
- Write report to file system
- Generate filename with timestamp
- Store in appropriate directory
- Create backup copy

### 6.2 Present to User

**Actions**:
- Display executive summary
- Provide full report link
- Offer interactive exploration
- Allow refinement and re-evaluation

### 6.3 Enable Iteration

**Options**:
- Adjust criteria weights
- Add/remove constraints
- Include/exclude specific models
- Request additional data sources
- Refine use case description

---

## Error Handling and Recovery

### Error Scenarios

1. **Missing Data**: Use available data, flag missing points, adjust confidence
2. **API Failures**: Retry with backoff, fall back to cached data, use alternative sources
3. **Conflicting Data**: Flag conflicts, apply source priority, document discrepancies
4. **Invalid Input**: Request clarification, provide examples, use defaults with warnings
5. **Timeout Errors**: Increase timeouts, optimize queries, provide partial results

### Recovery Strategies

- Graceful degradation of functionality
- Clear error messages to user
- Suggested workarounds
- Automatic retry with exponential backoff
- Fallback to cached or alternative data

---

## Performance Optimization

### Parallel Processing
- Query multiple MCP servers simultaneously
- Batch API calls where possible
- Parallelize independent data collection tasks

### Caching
- Cache model metadata (24-hour TTL)
- Cache benchmark data (7-day TTL)
- Cache search results (1-hour TTL)
- Implement cache invalidation on updates

### Progressive Loading
- Return preliminary results quickly
- Update with additional data as it arrives
- Allow user to proceed with partial information
- Background processing for comprehensive analysis

---

## Workflow Configuration

### Configurable Parameters

```json
{
  "workflow": {
    "maxCandidates": 10,
    "topRecommendations": 3,
    "timeout": 30000,
    "retryAttempts": 3,
    "cacheEnabled": true,
    "parallelQueries": true,
    "confidenceThreshold": 0.7
  },
  "criteria": {
    "defaultWeights": {
      "cost": 0.3,
      "performance": 0.4,
      "capabilities": 0.2,
      "community": 0.1
    }
  }
}
```

---

## Monitoring and Logging

### Events to Log
- User input received
- Criteria generated
- API calls made
- Data retrieved
- Evaluation performed
- Report generated
- Errors encountered

### Metrics to Track
- End-to-end latency
- Per-phase latency
- API call success rates
- Cache hit rates
- User satisfaction scores
- Error rates by type
