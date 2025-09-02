---
itemId: specs-ai-qms-ingestion
itemType: Software Item Spec
itemIsRelatedTo: KP-10644
itemFulfills: reqs-ai-qms-ingestion
---

# Software Item Specification: Automated QMS Document Ingestion and Rule Generation

## Overview

0. This Software Item Specification defines technical requirements for automated QMS document ingestion and rule generation within Ketryx EDMS.
1. The system implements AI-driven rule extraction from QMS documents to guide assistant behavior.
2. The specification ensures compliance with medical device regulatory standards.

## Scope

0. Automated processing of QMS documents from designated EDMS directories.
1. Two-pass AI rule generation with batch processing and intelligent filtering.
2. Feature flag controlled enablement at organization level.
3. Secure storage and access control for generated rules.
4. Comprehensive audit logging for regulatory compliance.

## System Architecture

### Document Processing Pipeline

0. The system monitors the "Quality Management System" directory in project EDMS.
1. Processing follows a queue-based architecture: Cron Job → Schedule Individual Jobs → Process Projects.
2. Documents are processed in batches with 40,000 token limits per batch.
3. Only DOCUMENT type items with uploaded files are processed.
4. Supported formats include PDF, DOCX, XLSX, and XML.

### Rule Generation Process

0. First pass extracts structured rules from document batches using AI analysis.
1. Second pass consolidates extracted rules into comprehensive rule sets.
2. Generated rules are stored as "ketryxAssistantRules.txt" in "Ketryx Assistant Rules" directory.
3. If generation fails, placeholder rules are created with error logging.

### Intelligent Rule Filtering

0. The `filterRelevantKetryxAssistantRules` function provides context-aware rule filtering.
1. Rules are filtered based on user query context using LLM analysis.
2. Filtering focuses on Configuration Item Rules, Process Rules, Risk Management Rules, and Compliance Rules.
3. Filtering is controlled by `llmFilterKetryxAssistantRules` feature flag.
4. If filtering fails, full rules are returned as fallback.

## Feature Flags

0. `llmUseKetryxAssistantRules` enables the assistant rules system for organizations.
1. `llmFilterKetryxAssistantRules` enables intelligent context-based rule filtering.
2. Feature flag status is checked during cron job processing.
3. Only organizations with enabled flags are processed.

## Processing Schedule

0. Automated processing runs via cron job (default: daily at 00:27 UTC).
1. Cron interval is configurable via `CRON_GENERATE_KETRYX_ASSISTANT_RULES` environment variable.
2. Processing follows a two-phase queue-based architecture for reliability.
3. Existing rules are detected to prevent duplicate generation.

## Rule Storage and Access

0. Rules are stored as JDF (JsonDocumentFormat) in EDMS documents.
1. Storage location is "Ketryx Assistant Rules" directory at project root.
2. Filename is standardized as "ketryxAssistantRules.txt".
3. AI Assistant and Agents have read access to rules via EDMS.
4. Rules are retrieved and converted to plain text for processing.

## Error Handling and Resilience

0. Queue-based architecture provides fault isolation between projects.
1. Database connection failures return empty string gracefully.
2. LLM processing failures create placeholder content with error context.
3. Rule filtering failures fall back to returning full unfiltered rules.
4. All errors are logged with appropriate severity levels.
5. Processing continues for other projects if individual projects fail.
6. Failed jobs can be retried independently without affecting other projects.

## Security and Authentication

0. All document access uses existing EDMS security controls.
1. Feature flag access requires organization-level permissions.
2. Rules are stored within project boundaries using EDMS access controls.
3. No additional authentication is required beyond existing EDMS security.

## Quality Assurance

0. Item record sampling provides formatting guidelines for rule generation.
1. Sample size is configurable (default: 60 records across different states).
2. Generated rules include project-specific item type configurations.
3. Rules are validated for JSON structure and completeness.

## Performance Considerations

0. Queue-based architecture prevents cron job stalling and enables parallel processing.
1. Document processing uses token counting to manage LLM request sizes.
2. Batch processing prevents memory exhaustion on large document sets.
3. Individual project jobs provide better resource management and progress tracking.
4. Cron job only performs lightweight scheduling, avoiding timeout issues.
5. Database queries are optimized with appropriate indexes and filters.

## Implementation Details

### Constants and File Locations

0. `QMS_DIRECTORY = 'Quality Management System'` - Source directory for QMS documents.
1. `ASSISTANT_RULES_DIRECTORY = 'Ketryx Assistant Rules'` - Target directory for generated rules.
2. `ASSISTANT_RULES_FILENAME = 'ketryxAssistantRules.txt'` - Standardized filename for rules.

### Key Functions

0. `processKetryxAssistantRulesGeneration()` - Main cron job scheduler in `ketryxAssistantRulesQueue.ts`.
1. `scheduleAssistantRulesGeneration()` - Schedules individual project processing jobs.
2. `processAssistantRulesQueue` - Queue processor for individual project jobs.
3. `generateKetryxAssistantRules()` - Core rule generation function in `aiUtils.ts`.
4. `filterRelevantKetryxAssistantRules()` - Context-aware rule filtering function in `aiUtils.ts`.
5. `sampleItemRecordsForFormatting()` - Provides formatting guidelines from existing records.

### Database Integration

0. Uses Prisma ORM for all database operations.
1. Creates DocumentFolder and Item records for rules storage.
2. Integrates with existing EDMS folder and document structure.
3. Maintains referential integrity with organization and project relationships.

## Rationale

0. Queue-based architecture eliminates cron job stalling issues and improves reliability.
1. This specification ensures reliable, scalable QMS document processing.
2. The implementation provides flexibility while maintaining regulatory compliance.
3. Context-aware filtering improves assistant relevance without sacrificing completeness.
4. Robust error handling ensures production reliability and auditability.
5. Individual job processing enables better fault isolation and progress tracking.
6. The solution supports compliance with 21 CFR Part 820, ISO 13485, and IEC 62304.
7. Integration with existing EDMS ensures security and access control consistency.
