```mermaid
erDiagram
    User {
        id userId PK "technical identifier"
        string email UK "nullable if Google-only"
        string passwordHash "one-way hash; email/password accounts only"
        string googleAccountId UK "Google accounts only"
        string mentorCode UK "permanent; HumanMentor role only"
    }

    UserRole {
        id userId PK,FK
        enum role PK "Learner | HumanMentor | PlatformAdministrator"
    }

    MentorshipRequest {
        id mentorshipRequestId PK "technical identifier"
        id learnerId FK
        id mentorId FK
        enum status "Pending | Accepted | Rejected"
    }

    Mentorship {
        id mentorshipId PK "technical identifier"
        id mentorshipRequestId FK,UK "must reference Accepted request"
        id learnerId FK
        id mentorId FK
        enum status "Active | Ended"
    }

    MentorshipConversation {
        id mentorshipConversationId PK "technical identifier"
        id mentorshipId FK,UK
    }

    MentorshipMessage {
        id mentorshipMessageId PK "technical identifier"
        id mentorshipConversationId FK
        id senderUserId FK
        integer messageOrder "unique within mentorshipConversationId"
        string content
    }

    AiMentorConversation {
        id aiMentorConversationId PK "technical identifier"
        id learnerId FK
    }

    AiMentorMessage {
        id aiMentorMessageId PK "technical identifier"
        id aiMentorConversationId FK
        enum senderType "Learner | AiMentor"
        integer messageOrder "unique within aiMentorConversationId"
        string content
    }

    LearningPath {
        id learningPathId PK "technical identifier"
        id learnerId FK
        string goal
        string track
        string currentKnowledge
    }

    Roadmap {
        id roadmapId PK "technical identifier"
        id learnerId FK
        id mentorshipId FK "nullable; XOR with learningPathId"
        id learningPathId FK,UK "nullable; XOR with mentorshipId"
    }

    Topic {
        id topicId PK "technical identifier"
        id roadmapId FK
        integer learningOrder "unique within roadmapId"
    }

    LearningMaterial {
        id learningMaterialId PK "technical identifier"
        id topicId FK
        string resourceUrl
    }

    ExternalService {
        id externalServiceId PK "technical identifier"
        string serviceName UK
    }

    LearnerExternalAccount {
        id learnerId PK,FK
        id externalServiceId PK,FK
        string externalAccountIdentifier
    }

    PracticalTask {
        id practicalTaskId PK "technical identifier"
        id topicId FK
        boolean isExternal
        id externalServiceId FK "nullable; required iff isExternal"
        string externalActivityIdentifier "nullable; required iff isExternal"
        enum completionStatus "NotCompleted | Completed"
    }

    Assessment {
        id assessmentId PK "technical identifier"
        id topicId FK
        enum assessmentType "Quiz | Task"
        string taskInstructions "nullable; required iff Task"
    }

    QuizQuestion {
        id quizQuestionId PK "technical identifier"
        id assessmentId FK
        string questionText
        enum questionType "SingleChoice | TrueFalse | MultipleSelect | ShortAnswer"
        number markAllocation
        string gradingCriteria "nullable; required iff ShortAnswer"
    }

    QuizAnswerChoice {
        id quizAnswerChoiceId PK "technical identifier"
        id quizQuestionId FK
        string choiceText
        boolean isCorrect
    }

    AssessmentSubmission {
        id assessmentSubmissionId PK "technical identifier"
        id assessmentId FK,UK
        string githubUrl "nullable; required iff Task"
    }

    QuizQuestionAnswer {
        id quizQuestionAnswerId PK "technical identifier"
        id assessmentSubmissionId FK
        id quizQuestionId FK,UK
        string writtenResponse "nullable; required iff ShortAnswer"
        number awardedMarks "auto-graded for choice; reviewed for ShortAnswer"
    }

    QuizSelectedChoice {
        id quizQuestionAnswerId PK,FK
        id quizAnswerChoiceId PK,FK
    }

    AssessmentReview {
        id assessmentReviewId PK "technical identifier"
        id assessmentSubmissionId FK,UK
        enum reviewerType "HumanMentor | AiMentor"
        id reviewerUserId FK "nullable; required iff HumanMentor"
        number marks
        string feedback
    }

    Project {
        id projectId PK "technical identifier"
        id topicId FK
    }

    ProjectSubmission {
        id projectSubmissionId PK "technical identifier"
        id projectId FK
        integer submissionNumber "unique within projectId"
        string githubUrl
    }

    ProjectReview {
        id projectReviewId PK "technical identifier"
        id projectSubmissionId FK,UK
        id mentorId FK
        enum outcome "Approved | NeedsRevision"
        string writtenFeedback
    }

    User ||--|{ UserRole : "holds"
    User ||--o{ MentorshipRequest : "submits as learner"
    User ||--o{ MentorshipRequest : "receives as mentor"
    MentorshipRequest ||--o| Mentorship : "activates"
    User ||--o{ Mentorship : "participates as learner"
    User ||--o{ Mentorship : "participates as mentor"
    Mentorship ||--|| MentorshipConversation : "has"
    MentorshipConversation ||--o{ MentorshipMessage : "contains"
    User ||--o{ MentorshipMessage : "sends"
    User ||--o{ AiMentorConversation : "owns as learner"
    AiMentorConversation ||--o{ AiMentorMessage : "contains"
    User ||--o{ LearningPath : "creates as learner"
    User ||--o{ Roadmap : "owns as learner"
    Mentorship |o--o{ Roadmap : "manages"
    LearningPath |o--o| Roadmap : "saves"
    Roadmap ||--|{ Topic : "contains"
    Topic ||--o{ LearningMaterial : "includes"
    User ||--o{ LearnerExternalAccount : "links as learner"
    ExternalService ||--o{ LearnerExternalAccount : "identifies account on"
    Topic ||--o{ PracticalTask : "includes"
    ExternalService |o--o{ PracticalTask : "verifies"
    Topic ||--o{ Assessment : "includes"
    Assessment ||--o{ QuizQuestion : "contains"
    QuizQuestion ||--o{ QuizAnswerChoice : "offers"
    Assessment ||--o| AssessmentSubmission : "receives"
    AssessmentSubmission ||--o{ QuizQuestionAnswer : "records"
    QuizQuestion ||--o| QuizQuestionAnswer : "answered by"
    QuizQuestionAnswer ||--o{ QuizSelectedChoice : "selects"
    QuizAnswerChoice ||--o| QuizSelectedChoice : "selected in"
    AssessmentSubmission ||--o| AssessmentReview : "evaluated by"
    User |o--o{ AssessmentReview : "reviews as mentor"
    Topic ||--o{ Project : "includes"
    Project ||--o{ ProjectSubmission : "receives"
    ProjectSubmission ||--o| ProjectReview : "receives official review"
    User ||--o{ ProjectReview : "officially reviews as mentor"
```

| Entity / Constraint | Scope & Logical Integrity Rule | Traceability |
| --- | --- | --- |
| `User` | Account supporting email/password and/or Google sign-in; every `User` must have at least one valid sign-in method: email/password (`email` and `passwordHash` both non-null, with `passwordHash` stored strictly as a one-way hash and never plaintext), Google (`googleAccountId` non-null; `email` optional unless email/password is also enabled), or both; unique permanent `mentorCode` assigned iff user holds `HumanMentor` role. | FR-01, FR-03, FR-06, NFR-01, EIR-01 |
| `UserRole` & Actor Eligibility | Multi-role assignment (`Learner`, `HumanMentor`, `PlatformAdministrator`); a single user may hold both `Learner` and `HumanMentor` simultaneously. **Role eligibility:** every `learnerId` reference (`MentorshipRequest`, `Mentorship`, `AiMentorConversation`, `LearningPath`, `Roadmap`, `LearnerExternalAccount`) requires the `User` to hold `Learner`; every `mentorId` reference (`MentorshipRequest`, `Mentorship`, `ProjectReview`) and `AssessmentReview.reviewerUserId` (when `reviewerType = HumanMentor`) requires the `User` to hold `HumanMentor`. | FR-02, FR-04, FR-06, FR-46, FR-47, NFR-04 |
| `MentorshipRequest` | Learner requests mentorship via valid `User.mentorCode`; starts `Pending` until Mentor accepts or rejects; code entry alone never creates a `Mentorship`. | FR-06 |
| `Mentorship` | Created only from an `Accepted` `MentorshipRequest` with matching `(learnerId, mentorId)`; Learner may hold multiple concurrent active Mentorships with different Mentors. | FR-05, FR-06, FR-13 |
| Active Mentorship Uniqueness & Ending | At most one `Mentorship` with `status = Active` may exist per `(learnerId, mentorId)` pair at a time; ended Mentorships remain recorded; a new Mentorship between the same pair is allowed only after the prior one is `Ended`; ending revokes former Mentor access while preserving Learner Roadmaps and history. | FR-11, FR-13, NFR-04, NFR-05 |
| `MentorshipConversation` & `MentorshipMessage` | Exactly one conversation per `Mentorship`; `senderUserId` must be either the `learnerId` or `mentorId` of that `Mentorship`; `messageOrder` is unique within `mentorshipConversationId` to preserve exchange order; former Mentor access revoked when `Mentorship.status = Ended`. | FR-08, FR-11, FR-12, FR-13, NFR-04, NFR-05 |
| `AiMentorConversation` & `AiMentorMessage` | Multiple private AI Mentor conversations per Learner, independent of Human Mentorship; `messageOrder` is unique within `aiMentorConversationId` to preserve conversation order; strictly separate from `MentorshipConversation`; never shared with Human Mentors; no routine admin browsing (narrow authorized operational/security/support access only). | FR-09, FR-10, FR-14, NFR-04, NFR-05, EIR-02 |
| `LearningPath` | Learner-defined `goal`, `track`, and `currentKnowledge`; multiple Learning Paths allowed per Learner; a new goal or track requires a new Learning Path; unsaved/regenerated AI Roadmap recommendations do not persist `Roadmap` records. | FR-15, FR-16, FR-17, FR-19, FR-20, FR-21, EIR-02 |
| `Roadmap` & Ownership XOR | Owned by one Learner (`learnerId`); **XOR:** exactly one of `mentorshipId` or `learningPathId` must be non-null; `Roadmap.learnerId` must equal `Mentorship.learnerId` or `LearningPath.learnerId`; `learningPathId` is unique (at most one saved Roadmap per `LearningPath`); Mentor-prepared Roadmaps using AI Assistant remain Mentor-managed (`mentorshipId` non-null). | FR-07, FR-17, FR-18, FR-20, FR-26, FR-27, NFR-05 |
| `Topic` | One or more Topics per `Roadmap`, ordered by unique `(roadmapId, learningOrder)`. | FR-22, FR-23, FR-28 |
| `LearningMaterial` | Zero or more optional Learning Materials per `Topic`, each accessed via `resourceUrl`. | FR-23, FR-24, FR-29 |
| `ExternalService` & `LearnerExternalAccount` | Generic supported external verification services; composite PK `(learnerId, externalServiceId)` enforces at most one linked external account identifier per Learner per service. | FR-42, FR-43, EIR-03 |
| `PracticalTask` & Completion Rules | When `isExternal = true`, `externalServiceId` and `externalActivityIdentifier` are required and `completionStatus` may become `Completed` only after successful external API verification using the Learner's `LearnerExternalAccount`. When `isExternal = false`, external fields are null and `Completed` may be set only by the responsible Human Mentor (Mentor-managed Roadmap) or the Learner (Learner-owned Roadmap). | FR-23, FR-41, FR-42, FR-43, NFR-05, NFR-06, EIR-03 |
| `Assessment` & Mentor Requirement | Supports `Quiz` and `Task`; `taskInstructions` required iff `Task` (provided by responsible Mentor on Mentor-managed Roadmaps, AI Assistant on Learner-owned Roadmaps); every `Assessment` on a Mentor-managed Roadmap is required for the Learner; Topics are not required to contain an Assessment. | FR-23, FR-25, FR-31, FR-32 |
| `QuizQuestion` & `QuizAnswerChoice` | Valid only for `Assessment.assessmentType = Quiz`; supports `SingleChoice`, `TrueFalse`, `MultipleSelect`, `ShortAnswer` on both Roadmap types; `gradingCriteria` required iff `ShortAnswer` and null for choice-based questions; `QuizAnswerChoice` applies only to choice-based questions (1 correct choice for `SingleChoice`/`TrueFalse`, 1+ for `MultipleSelect`). | FR-33, FR-34, FR-35, FR-38 |
| `AssessmentSubmission` | `assessmentId` is unique (at most one submission per `Assessment`); submitting Learner is `Roadmap.learnerId`; `githubUrl` required iff `Assessment.assessmentType = Task` and null for `Quiz`. | FR-30, FR-40, NFR-05, NFR-06 |
| `QuizQuestionAnswer` & `QuizSelectedChoice` | Valid only for `Quiz` submissions; `QuizQuestion.assessmentId` must equal `AssessmentSubmission.assessmentId`; `QuizAnswerChoice.quizQuestionId` must equal `QuizQuestionAnswer.quizQuestionId`; `writtenResponse` required iff `ShortAnswer`; choice-based answers use `QuizSelectedChoice` (at most 1 choice for `SingleChoice`/`TrueFalse`) and are auto-graded on both Roadmap types; `MultipleSelect` awards full `markAllocation` iff all correct choices and no incorrect choices are selected, else `0`. | FR-36, FR-37, FR-39, FR-48 |
| `AssessmentReview` | At most one review per `AssessmentSubmission` (`assessmentSubmissionId` unique); routed by Roadmap ownership: Mentor-managed requires `reviewerType = HumanMentor` with `reviewerUserId = Mentorship.mentorId` (Mentor evaluates Short Answer/Task and provides marks/feedback alongside auto-graded choice marks); Learner-owned requires `reviewerType = AiMentor` with `reviewerUserId = null`; results visible to the submitting Learner. | FR-46, FR-48, FR-49, FR-51, FR-52, NFR-05, NFR-06, EIR-02 |
| `Project` & `ProjectSubmission` | Belongs to a `Topic`; required for the Learner iff parent Roadmap is Mentor-managed; Learner (`Roadmap.learnerId`) may submit multiple separate `ProjectSubmission` records via `githubUrl`, ordered by unique `(projectId, submissionNumber)` to identify prior and latest submissions; all submissions and reviews are preserved and visible to the Learner. | FR-23, FR-25, FR-44, FR-45, FR-52, NFR-05, NFR-06 |
| `ProjectReview` | Official Human Mentor review only; at most one per `ProjectSubmission` (`projectSubmissionId` unique); permitted **only** when parent Roadmap is Mentor-managed (`mentorshipId` non-null) and `mentorId = Mentorship.mentorId` while Mentorship is active; `outcome` is `Approved` or `NeedsRevision`; strictly forbidden on Learner-owned Roadmaps and never performed by AI. | FR-47, FR-50, FR-52, NFR-04, NFR-05, NFR-06, EIR-02 |
| History Preservation & Status Separation | Changing Roadmap items, updating progress, or ending a Mentorship must never overwrite or delete `PracticalTask.completionStatus`, `AssessmentSubmission`, `AssessmentReview`, `ProjectSubmission`, or `ProjectReview`; UI/responses must present PracticalTask completion, Assessment evaluation, Project submission/official review, and non-official AI guidance separately. | FR-13, NFR-05, NFR-06 |
| Non-Schema Policy & Behavioral Rules | Not modeled as ERD entities: (1) AI privacy and external-transmission notices to Learners/Mentors and per-request context selection rules; (2) AI data retention and deletion rules (no fixed retention period is approved in the MVP); (3) runtime calculation rule for 100% Topic completion triggering an optional AI Quiz offer on Learner-owned Roadmaps. | NFR-02, NFR-03; System Design §2 |
| Unresolved Persistence Details | **Unresolved in approved sources:** (1) Platform Administrator account-management/operational attributes (FR-04); (2) descriptive/display attributes (e.g., titles/descriptions) for `Roadmap`, `Topic`, `LearningMaterial`, `PracticalTask`, `Assessment`, `Project`, and user profile fields; (3) whether Human Mentor AI Assistant interactions are persisted (FR-18); (4) whether non-official Project AI guidance is stored separately or delivered within `AiMentorConversation` (FR-47, NFR-06). | FR-04, FR-18, FR-47, NFR-06 |
