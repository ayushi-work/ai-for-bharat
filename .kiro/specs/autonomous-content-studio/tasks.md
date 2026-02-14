# Implementation Plan: Autonomous Content Studio

## Overview

This implementation plan breaks down the Autonomous Content Studio into discrete coding tasks. The system will be built in Go, following the agent-native architecture with A2A protocol-based communication. The implementation follows a bottom-up approach: first establishing core data structures and the A2A protocol, then implementing individual agents, and finally wiring everything together through the orchestrator.

## Tasks

- [ ] 1. Set up project structure and core data models
  - Create Go module with appropriate directory structure (cmd/, internal/, pkg/)
  - Define core data structures: WorkflowInput, WorkflowOutput, SharedState, ExecutionContext
  - Define memory models: ContentMemory, PerformanceMemory
  - Define content models: ContentMetadata, NarrativeOption, ScheduleEntry, TraceEntry
  - Implement JSON serialization/deserialization for all data models
  - Set up testing framework (testing package, testify for assertions)
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 7.3, 8.1, 8.2_

- [ ]* 1.1 Write property test for data model serialization
  - **Property 38: Memory Serialization Round-Trip**
  - **Validates: Requirements 8.4**

- [ ] 2. Implement A2A Protocol
  - [ ] 2.1 Create A2A message structures
    - Define A2AMessage struct with all required fields (message_id, sender_agent_id, recipient_agent_id, timestamp, schema_version, execution_context, payload)
    - Define A2AResponse struct with status, payload, and error_details
    - Implement JSON marshaling/unmarshaling for messages
    - _Requirements: 6.1, 6.2, 6.3, 6.4_
  
  - [ ]* 2.2 Write property test for A2A message structure
    - **Property 27: A2A Message Structure Completeness**
    - **Validates: Requirements 6.1, 6.2, 6.3, 6.4**
  
  - [ ] 2.3 Create message validation functions
    - Implement validateMessage() to check required fields are non-null
    - Implement validateResponse() to check response schema
    - Implement schema version compatibility checking
    - _Requirements: 6.4, 11.4_
  
  - [ ]* 2.4 Write property test for schema version compatibility
    - **Property 50: Schema Version Compatibility**
    - **Validates: Requirements 11.4**

- [ ] 3. Implement Agent Registry
  - [ ] 3.1 Create agent registry structures
    - Define AgentRegistryEntry struct with agent metadata
    - Define AgentRegistry struct with agent map
    - Implement RegisterAgent() and GetAgent() methods
    - Implement GetExecutionOrder() to determine agent sequence based on dependencies
    - _Requirements: 11.1, 11.3_
  
  - [ ]* 3.2 Write property test for agent registry
    - **Property 48: Agent Registry Existence**
    - **Property 49: Dynamic Agent Discovery**
    - **Validates: Requirements 11.1, 11.3**

- [ ] 4. Implement Memory Management
  - [ ] 4.1 Create memory storage and retrieval
    - Implement LoadContentMemory() to read from JSON file
    - Implement SaveContentMemory() to persist to JSON file
    - Implement AddTopic() and AddHook() methods with idempotency
    - Implement IsTopicRecent() to check topic usage within time window
    - Create .content_studio/memory/ directory structure
    - _Requirements: 8.1, 8.3, 8.4, 8.5_
  
  - [ ]* 4.2 Write property test for memory idempotency
    - **Property 39: Memory Update Idempotency**
    - **Validates: Requirements 8.5**
  
  - [ ] 4.3 Implement performance memory (mocked for MVP)
    - Create PerformanceMemory struct with placeholder data
    - Implement LoadPerformanceMemory() with default values
    - _Requirements: 8.2_

- [ ] 5. Implement Ideation Agent
  - [ ] 5.1 Create Ideation Agent structure and interface
    - Define IdeationAgent struct
    - Implement Process() method accepting A2AMessage and returning A2AResponse
    - Define agent ID constant
    - _Requirements: 2.1, 2.6_
  
  - [ ] 5.2 Implement narrative generation logic
    - Implement generateNarrativeOptions() to create 3-5 theme options
    - Implement selectBestTheme() to choose theme based on novelty and relevance
    - Implement checkAgainstMemory() to avoid recent topics
    - Generate supporting angles for selected theme
    - Generate selection rationale
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_
  
  - [ ]* 5.3 Write property tests for Ideation Agent
    - **Property 6: Multiple Narrative Options**
    - **Property 7: Topic Novelty**
    - **Property 8: Single Theme Selection**
    - **Property 9: Supporting Angles Presence**
    - **Property 10: Selection Rationale**
    - **Property 11: Ideation A2A Response**
    - **Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5, 2.6**
  
  - [ ]* 5.4 Write unit tests for edge cases
    - Test with empty content memory
    - Test with all topics marked as recent
    - Test with very short content ideas

- [ ] 6. Implement Core Content Agent
  - [ ] 6.1 Create Core Content Agent structure
    - Define CoreContentAgent struct
    - Implement Process() method accepting A2AMessage and returning A2AResponse
    - Define agent ID constant
    - _Requirements: 3.1, 3.5_
  
  - [ ] 6.2 Implement content generation logic
    - Implement generateHook() for attention-grabbing opening
    - Implement generateExplanation() for core concept (2-3 paragraphs)
    - Implement generateExamples() for concrete illustrations
    - Implement generateCTA() for call-to-action
    - Assemble full content from sections
    - Generate ContentMetadata with all sections and word count
    - _Requirements: 3.2, 3.3, 3.4_
  
  - [ ]* 6.3 Write property tests for Core Content Agent
    - **Property 12: Content Generation**
    - **Property 13: Content Structure Completeness**
    - **Property 14: Long-Form Length**
    - **Property 15: Metadata Presence**
    - **Property 16: Core Content A2A Response**
    - **Validates: Requirements 3.1, 3.2, 3.3, 3.4, 3.5**
  
  - [ ]* 6.4 Write unit tests for content structure
    - Test hook generation with various themes
    - Test explanation generation with different angles
    - Test word count boundaries (800-1200 words)

- [ ] 7. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 8. Implement Repurposing Agent
  - [ ] 8.1 Create Repurposing Agent structure
    - Define RepurposingAgent struct
    - Implement Process() method accepting A2AMessage and returning A2AResponse
    - Define agent ID constant
    - _Requirements: 4.1, 4.6_
  
  - [ ] 8.2 Define platform constraints
    - Create PlatformConstraints struct
    - Define constraints for LinkedIn (3000 chars, professional tone)
    - Define constraints for Twitter (280 chars per tweet, concise tone)
    - Define constraints for Instagram (2200 chars, casual tone)
    - Define constraints for Blog (800-1200 words, detailed tone)
    - Implement getPlatformConstraints() lookup function
    - _Requirements: 4.2, 4.3, 4.4_
  
  - [ ] 8.3 Implement content adaptation logic
    - Implement adaptForPlatform() for each platform
    - Implement applyLengthConstraints() to truncate/summarize content
    - Implement applyToneAdjustment() to modify language style
    - Implement applyFormatting() for hashtags, line breaks, emojis
    - Generate AdaptationMetadata for each adaptation
    - _Requirements: 4.2, 4.3, 4.4, 4.5_
  
  - [ ]* 8.4 Write property tests for Repurposing Agent
    - **Property 17: Complete Platform Coverage**
    - **Property 18: Platform Length Constraints**
    - **Property 19: Tone Metadata Recording**
    - **Property 20: Platform-Specific Formatting**
    - **Property 21: Adaptation Bundle Completeness**
    - **Property 22: Repurposing A2A Response**
    - **Validates: Requirements 4.1, 4.2, 4.3, 4.4, 4.5, 4.6**
  
  - [ ]* 8.5 Write unit tests for platform adaptations
    - Test LinkedIn adaptation with long content
    - Test Twitter thread generation
    - Test Instagram emoji insertion
    - Test length constraint enforcement

- [ ] 9. Implement Distribution Agent
  - [ ] 9.1 Create Distribution Agent structure
    - Define DistributionAgent struct
    - Implement Process() method accepting A2AMessage and returning A2AResponse
    - Define agent ID constant
    - _Requirements: 5.1, 5.5_
  
  - [ ] 9.2 Implement scheduling logic
    - Define optimal time windows for each platform
    - Implement getOptimalTime() to select time within platform window
    - Implement determineSequenceOrder() for cross-platform sequencing (blog first, then LinkedIn, then Twitter, then Instagram)
    - Generate ScheduleEntry for each platform with timestamp and rationale
    - _Requirements: 5.2, 5.3, 5.4_
  
  - [ ]* 9.3 Write property tests for Distribution Agent
    - **Property 23: Schedule Generation**
    - **Property 24: Time-of-Day Optimization**
    - **Property 25: Platform Sequencing**
    - **Property 26: Distribution A2A Response**
    - **Validates: Requirements 5.1, 5.2, 5.3, 5.4, 5.5**
  
  - [ ]* 9.4 Write unit tests for scheduling rules
    - Test LinkedIn weekday morning scheduling
    - Test Twitter multiple-times-daily scheduling
    - Test blog-first sequencing rule
    - Test weekend vs weekday logic

- [ ] 10. Implement Orchestrator Core
  - [ ] 10.1 Create Orchestrator structure
    - Define Orchestrator struct with agent registry, memory, and state
    - Implement NewOrchestrator() constructor
    - Initialize agent registry with all four agents
    - Load content memory and performance memory
    - _Requirements: 7.1, 11.1_
  
  - [ ] 10.2 Implement input validation
    - Implement validateInput() for WorkflowInput
    - Check content idea is non-empty
    - Check platforms list contains 2-3 valid platforms
    - Return validation errors with clear messages
    - _Requirements: 1.2_
  
  - [ ]* 10.3 Write property test for input validation
    - **Property 2: Platform Validation**
    - **Validates: Requirements 1.2**

- [ ] 11. Implement Orchestrator Workflow Execution
  - [ ] 11.1 Implement workflow execution method
    - Create ExecuteWorkflow() method accepting WorkflowInput
    - Initialize SharedState with input data
    - Initialize execution trace
    - Generate unique workflow ID
    - _Requirements: 1.1, 1.5, 7.3_
  
  - [ ] 11.2 Implement agent execution loop
    - Get agent execution order from registry
    - For each agent in sequence:
      - Prepare A2A message with current state
      - Call agent's Process() method
      - Validate agent response
      - Update shared state with agent output
      - Add trace entry
      - Persist memory updates
    - _Requirements: 7.1, 7.2, 7.4, 7.6, 12.1, 12.3_
  
  - [ ] 11.3 Implement A2A message routing
    - Implement sendA2AMessage() to route messages to agents
    - Implement validateAgentOutput() to check response schema
    - Implement updateSharedState() to merge agent outputs
    - _Requirements: 6.5, 7.2, 7.3, 7.4_
  
  - [ ]* 11.4 Write property tests for orchestrator execution
    - **Property 1: Workflow Initiation**
    - **Property 5: Autonomous Execution**
    - **Property 29: Deterministic Agent Execution Order**
    - **Property 30: Output Validation Before Proceeding**
    - **Property 31: Shared State Accumulation**
    - **Property 32: State Propagation in Messages**
    - **Property 51: Sequential Agent Execution**
    - **Validates: Requirements 1.1, 1.5, 7.1, 7.2, 7.3, 7.4, 12.1, 12.3**

- [ ] 12. Implement Orchestrator State and Memory Management
  - [ ] 12.1 Implement state management methods
    - Implement updateSharedState() to merge new data
    - Implement getStateSnapshot() for A2A message context
    - Validate state after each update
    - _Requirements: 7.3, 7.4_
  
  - [ ] 12.2 Implement memory persistence
    - Implement persistMemory() to save content memory after workflow
    - Add new topics and hooks to memory
    - Ensure idempotent memory updates
    - Handle persistence errors with retry logic
    - _Requirements: 7.5, 8.5_
  
  - [ ]* 12.3 Write property tests for memory management
    - **Property 33: Memory Persistence**
    - **Property 35: Content Memory Structure**
    - **Property 37: Memory Inclusion in Agent Messages**
    - **Validates: Requirements 7.5, 8.1, 8.3**

- [ ] 13. Implement Orchestrator Output and Observability
  - [ ] 13.1 Implement execution trace logging
    - Create addTraceEntry() method
    - Log agent name, timestamp, input/output summaries, execution time
    - Ensure trace contains entry for each executed agent
    - _Requirements: 7.6, 7.7, 10.2, 10.3, 10.4_
  
  - [ ] 13.2 Implement output assembly
    - Assemble WorkflowOutput from shared state
    - Include core content, platform adaptations, schedule, and trace
    - Add workflow ID and timestamp
    - Serialize output to JSON
    - _Requirements: 1.4, 7.7_
  
  - [ ]* 13.3 Write property tests for observability
    - **Property 4: Complete Output Structure**
    - **Property 34: Complete Execution Trace**
    - **Property 45: Trace Entry Timestamps**
    - **Property 46: Trace Entry Summaries**
    - **Validates: Requirements 1.4, 7.6, 7.7, 10.2, 10.3, 10.4**

- [ ] 14. Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 15. Implement Error Handling
  - [ ] 15.1 Create error types and structures
    - Define ErrorResponse struct
    - Define error type constants (input_validation, agent_execution, communication, state_management)
    - Implement error response formatting
    - _Requirements: 12.5_
  
  - [ ] 15.2 Implement agent execution error handling
    - Wrap agent Process() calls in error handling
    - Implement retry logic with exponential backoff (up to 2 retries)
    - Halt workflow on agent failure after retries exhausted
    - Add error details to execution trace
    - _Requirements: 9.5, 12.5_
  
  - [ ] 15.3 Implement validation error handling
    - Handle input validation errors before workflow starts
    - Handle agent output validation errors during execution
    - Handle A2A message schema validation errors
    - Return appropriate error responses
    - _Requirements: 7.2, 12.5_
  
  - [ ]* 15.4 Write property test for error handling
    - **Property 52: Failure Halts Workflow**
    - **Validates: Requirements 12.5**
  
  - [ ]* 15.5 Write unit tests for error scenarios
    - Test input validation errors
    - Test agent execution failures
    - Test retry logic
    - Test error response formatting

- [ ] 16. Implement Agent Isolation and Idempotency
  - [ ]* 16.1 Write property tests for agent isolation
    - **Property 40: Agent Independence**
    - **Property 41: Agent Idempotency**
    - **Property 42: Context Completeness**
    - **Property 43: Agent Statelessness**
    - **Property 44: Isolated Agent Retry**
    - **Validates: Requirements 9.1, 9.2, 9.3, 9.4, 9.5**
  
  - [ ]* 16.2 Write unit tests for idempotency
    - Test each agent with identical inputs multiple times
    - Verify outputs are equivalent
    - Test orchestrator retry without re-running previous agents

- [ ] 17. Implement CLI Interface
  - [ ] 17.1 Create command-line interface
    - Create main.go in cmd/content-studio/
    - Parse command-line arguments (content-idea, platforms, audience)
    - Validate CLI inputs
    - _Requirements: 1.1, 1.2, 1.3_
  
  - [ ] 17.2 Implement workflow execution from CLI
    - Initialize Orchestrator
    - Create WorkflowInput from CLI args
    - Execute workflow
    - Handle errors and display error messages
    - Display workflow output (core content, adaptations, schedule, trace)
    - Save output to file in .content_studio/workflows/{workflow_id}/
    - _Requirements: 1.4, 1.5_
  
  - [ ]* 17.3 Write integration tests for CLI
    - Test end-to-end workflow execution
    - Test with various platform combinations
    - Test with and without audience
    - Test error handling from CLI

- [ ] 18. Implement LLM Integration (Mocked for Testing)
  - [ ] 18.1 Create LLM client interface
    - Define LLMClient interface with Generate() method
    - Create MockLLMClient for testing with deterministic outputs
    - Create RealLLMClient for production (placeholder for AWS Bedrock or similar)
    - _Requirements: 2.1, 3.1, 4.1, 5.1_
  
  - [ ] 18.2 Integrate LLM client into agents
    - Pass LLMClient to each agent constructor
    - Use LLM client in narrative generation (Ideation Agent)
    - Use LLM client in content generation (Core Content Agent)
    - Use LLM client in content adaptation (Repurposing Agent)
    - Use mock client in all tests
    - _Requirements: 2.1, 3.1, 4.1_

- [ ] 19. Final Integration and Testing
  - [ ]* 19.1 Write end-to-end integration tests
    - Test complete workflow with all agents
    - Test with different content ideas and platform combinations
    - Verify all outputs are generated correctly
    - Verify execution trace is complete
    - Verify memory is persisted
    - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5_
  
  - [ ]* 19.2 Write property tests for remaining properties
    - **Property 3: Optional Audience Propagation**
    - **Property 28: Orchestrator Message Coordination**
    - **Property 36: Performance Memory Structure**
    - **Property 47: Platform Format Compliance**
    - **Validates: Requirements 1.3, 6.5, 6.6, 8.2, 10.5**
  
  - [ ] 19.3 Create example workflows
    - Create example content ideas in examples/ directory
    - Run workflows and save outputs
    - Document expected outputs
    - _Requirements: 1.1, 1.4_

- [ ] 20. Final Checkpoint - Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at key milestones
- Property tests validate universal correctness properties (minimum 100 iterations each)
- Unit tests validate specific examples and edge cases
- LLM integration is mocked in tests to ensure deterministic behavior
- The implementation follows a bottom-up approach: data models → protocol → agents → orchestrator → CLI
- All agents must be stateless and idempotent
- All inter-agent communication must flow through the Orchestrator via A2A protocol
