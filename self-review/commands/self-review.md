---
description: PR 준비를 위한 자동화된 품질 보증 셀프 리뷰 워크플로우
argument-hint: 선택적 기능 설명
---

# Self-Review Workflow

Use the Task tool to launch the `self-review:orchestrator` agent to guide you through a comprehensive PR preparation workflow.

The orchestrator will:
1. Gather and clarify your requirements
2. Create a feature branch automatically
3. Explore the codebase using code-explorer agents
4. Ask clarifying questions before implementation
5. Design architecture using code-architect agents
6. Implement features and create commits
7. Run comprehensive reviews using all 6 pr-review-toolkit agents
8. Iteratively fix issues and re-review until quality is acceptable
9. Prepare PR summary and next steps

**Initial request**: $ARGUMENTS

Launch the orchestrator agent now.
