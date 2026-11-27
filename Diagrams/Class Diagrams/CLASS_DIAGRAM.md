```mermaid
classDiagram
    class IGoogleAuthProvider {
        <<interface>>
        +authenticateGoogleToken(string token) string
    }

    class IExternalAiService {
        <<interface>>
        +generateRoadmapRecommendation(string goal, string track, string currentKnowledge) Roadmap
        +suggestLearningMaterials(string context) List~string~
        +generateAiResponse(string prompt, string context) string
        +evaluateAssessment(AssessmentSubmission submission, Map~quizQuestionId, number~ outShortAnswerMarks) AssessmentReview
    }

    class IExternalVerificationService {
        <<interface>>
        +verifyTaskCompletion(string serviceName, string accountIdentifier, string activityIdentifier) boolean
    }

    class User {
        -id userId
        -string email
        -string passwordHash
        -string googleAccountId
        -string mentorCode
        +registerWithEmail(string email, string rawPassword) User
        +registerWithGoogle(IGoogleAuthProvider authProvider, string token) User
        +signIn(string email, string credential, IGoogleAuthProvider authProvider) boolean
        +assignRole(User adminActor, enum role) UserRole
        +hasRole(enum role) boolean
        +assignMentorCode() string
        -hashPassword(string rawPassword) string
    }

    class UserRole {
        -id userId
        -enum role
    }

    class MentorshipRequest {
        -id mentorshipRequestId
        -id learnerId
        -id mentorId
        -enum status
        +createRequestByCode(User learner, string mentorCode) MentorshipRequest
        +accept(User mentor) Mentorship
        +reject(User mentor) void
    }

    class Mentorship {
        -id mentorshipId
        -id mentorshipRequestId
        -id learnerId
        -id mentorId
        -enum status
        +end() void
        +hasActiveMentorAccess(id mentorId) boolean
    }

    class MentorshipConversation {
        -id mentorshipConversationId
        -id mentorshipId
        +sendMessage(User sender, string content) MentorshipMessage
    }

    class MentorshipMessage {
        -id mentorshipMessageId
        -id mentorshipConversationId
        -id senderUserId
        -integer messageOrder
        -string content
    }

    class AiMentorConversation {
        -id aiMentorConversationId
        -id learnerId
        +presentAiPrivacyNotice() void
        +sendAiPrompt(string prompt, IExternalAiService aiService) AiMentorMessage
        +requestAiDataDeletion() void
    }

    class AiMentorMessage {
        -id aiMentorMessageId
        -id aiMentorConversationId
        -enum senderType
        -integer messageOrder
        -string content
    }

    class LearningPath {
        -id learningPathId
        -id learnerId
        -string goal
        -string track
        -string currentKnowledge
        +recommendRoadmap(IExternalAiService aiService) Roadmap
        +regenerateRecommendation(IExternalAiService aiService) Roadmap
        +saveRoadmap(Roadmap recommendation) Roadmap
    }

    class Roadmap {
        -id roadmapId
        -id learnerId
        -id mentorshipId
        -id learningPathId
        +prepareWithAiAssistant(User mentor, Mentorship mentorship, IExternalAiService aiService) Roadmap
        +suggestMaterialsWithAiAssistant(User mentor, Mentorship mentorship, string context, IExternalAiService aiService) List~string~
        +addTopic(integer learningOrder) Topic
        +orderTopics() List~Topic~
        #validateOwnershipXor() boolean
    }

    class Topic {
        -id topicId
        -id roadmapId
        -integer learningOrder
        +calculateCompletionRate() number
        +isEligibleForTopicAiQuiz() boolean
    }

    class LearningMaterial {
        -id learningMaterialId
        -id topicId
        -string resourceUrl
    }

    class ExternalService {
        -id externalServiceId
        -string serviceName
    }

    class LearnerExternalAccount {
        -id learnerId
        -id externalServiceId
        -string externalAccountIdentifier
        +linkExternalAccount(User learner, ExternalService service, string identifier) LearnerExternalAccount
    }

    class PracticalTask {
        -id practicalTaskId
        -id topicId
        -boolean isExternal
        -id externalServiceId
        -string externalActivityIdentifier
        -enum completionStatus
        +verifyViaExternalApi(LearnerExternalAccount account, IExternalVerificationService verifier) boolean
        +markCompleted(User actor) void
    }

    class Assessment {
        -id assessmentId
        -id topicId
        -enum assessmentType
        -string taskInstructions
        +submitQuiz(User learner, Map~id, string~ writtenResponses, Map~id, id[]~ selectedChoiceIdsByQuestion) AssessmentSubmission
        +submitTask(User learner, string githubUrl) AssessmentSubmission
    }

    class QuizQuestion {
        -id quizQuestionId
        -id assessmentId
        -string questionText
        -enum questionType
        -number markAllocation
        -string gradingCriteria
        +autoGradeChoice(List~QuizAnswerChoice~ selectedChoices) number
        +scoreMultipleSelectAllOrNothing(List~QuizAnswerChoice~ selectedChoices) number
    }

    class QuizAnswerChoice {
        -id quizAnswerChoiceId
        -id quizQuestionId
        -string choiceText
        -boolean isCorrect
    }

    class AssessmentSubmission {
        -id assessmentSubmissionId
        -id assessmentId
        -string githubUrl
        +submitMentorReview(User mentor, Map~quizQuestionId, number~ shortAnswerMarks, number taskMarks, string feedback) AssessmentReview
        +evaluateWithAi(IExternalAiService aiService) AssessmentReview
    }

    class QuizQuestionAnswer {
        -id quizQuestionAnswerId
        -id assessmentSubmissionId
        -id quizQuestionId
        -string writtenResponse
        -number awardedMarks
        ~evaluateShortAnswer(number marks) void
    }

    class QuizSelectedChoice {
        -id quizQuestionAnswerId
        -id quizAnswerChoiceId
    }

    class AssessmentReview {
        -id assessmentReviewId
        -id assessmentSubmissionId
        -enum reviewerType
        -id reviewerUserId
        -number marks
        -string feedback
        #routeReviewer(Roadmap roadmap) enum
    }

    class Project {
        -id projectId
        -id topicId
        +submitVersion(User learner, string githubUrl) ProjectSubmission
    }

    class ProjectSubmission {
        -id projectSubmissionId
        -id projectId
        -integer submissionNumber
        -string githubUrl
        +submitOfficialReview(User mentor, enum outcome, string writtenFeedback) ProjectReview
        +requestNonOfficialAiGuidance(IExternalAiService aiService) string
    }

    class ProjectReview {
        -id projectReviewId
        -id projectSubmissionId
        -id mentorId
        -enum outcome
        -string writtenFeedback
    }

    User "1" *-- "1..*" UserRole : holds
    User "1" --> "0..*" MentorshipRequest : submits as learner
    User "1" --> "0..*" MentorshipRequest : receives as mentor
    MentorshipRequest "1" --> "0..1" Mentorship : activates
    User "1" --> "0..*" Mentorship : participates as learner
    User "1" --> "0..*" Mentorship : participates as mentor
    Mentorship "1" *-- "1" MentorshipConversation : has
    MentorshipConversation "1" *-- "0..*" MentorshipMessage : contains
    User "1" --> "0..*" MentorshipMessage : sends
    User "1" o-- "0..*" AiMentorConversation : owns as learner
    AiMentorConversation "1" *-- "0..*" AiMentorMessage : contains
    User "1" o-- "0..*" LearningPath : creates as learner
    User "1" o-- "0..*" Roadmap : owns as learner
    Mentorship "0..1" o-- "0..*" Roadmap : manages
    LearningPath "0..1" o-- "0..1" Roadmap : saves
    Roadmap "1" *-- "1..*" Topic : contains
    Topic "1" *-- "0..*" LearningMaterial : includes
    User "1" --> "0..*" LearnerExternalAccount : links as learner
    ExternalService "1" --> "0..*" LearnerExternalAccount : identifies account on
    Topic "1" *-- "0..*" PracticalTask : includes
    ExternalService "0..1" --> "0..*" PracticalTask : verifies
    Topic "1" *-- "0..*" Assessment : includes
    Assessment "1" *-- "0..*" QuizQuestion : contains
    QuizQuestion "1" *-- "0..*" QuizAnswerChoice : offers
    Assessment "1" o-- "0..1" AssessmentSubmission : receives
    AssessmentSubmission "1" *-- "0..*" QuizQuestionAnswer : records
    QuizQuestion "1" --> "0..1" QuizQuestionAnswer : answered by
    QuizQuestionAnswer "1" *-- "0..*" QuizSelectedChoice : selects
    QuizAnswerChoice "1" --> "0..1" QuizSelectedChoice : selected in
    AssessmentSubmission "1" o-- "0..1" AssessmentReview : evaluated by
    User "0..1" --> "0..*" AssessmentReview : reviews as mentor
    Topic "1" *-- "0..*" Project : includes
    Project "1" o-- "0..*" ProjectSubmission : receives
    ProjectSubmission "1" o-- "0..1" ProjectReview : receives official review
    User "1" --> "0..*" ProjectReview : officially reviews as mentor

    User ..> IGoogleAuthProvider : authenticates via
    AiMentorConversation ..> IExternalAiService : sends prompts via
    LearningPath ..> IExternalAiService : requests recommendations via
    Roadmap ..> IExternalAiService : prepares with AI assistant via
    AssessmentSubmission ..> IExternalAiService : evaluates Learner-owned via
    ProjectSubmission ..> IExternalAiService : requests non-official guidance via
    PracticalTask ..> IExternalVerificationService : verifies completion via
```

| Class / Interface | Core Methods & Domain Responsibility | Traceability |
| --- | --- | --- |
| `User` | `registerWithEmail()`, `registerWithGoogle()`, `signIn()`, `assignRole()`, `hasRole()`, `assignMentorCode()`, `hashPassword()`; supports email/password (one-way hashed) and/or Google sign-in, multi-role membership, and permanent unique Mentor Code for Human Mentors; role changes via `assignRole()` require an authorized `PlatformAdministrator`, while initial account registration handles approved default roles. | FR-01, FR-02, FR-03, FR-04, FR-06, NFR-01, NFR-04, EIR-01 |
| `UserRole` | Multi-role assignment (`Learner`, `HumanMentor`, `PlatformAdministrator`); supports simultaneous `Learner` and `HumanMentor` roles per account; subsequent role changes require an authorized `PlatformAdministrator`, while initial account registration handles approved default roles. | FR-02, FR-03, FR-04, NFR-04 |
| `MentorshipRequest` | `createRequestByCode()`, `accept()`, `reject()`; pending request workflow via valid Mentor Code without auto-activation; both `accept(mentor)` and `reject(mentor)` require authenticated `mentor` holding `HumanMentor`, `mentor.userId == this.mentorId`, and `this.status == Pending`; `accept(mentor)` transitions status to `Accepted` and creates a `Mentorship` with matching `(learnerId, mentorId)` subject to the single-active-Mentorship rule (`FR-13`), whereas `reject(mentor)` transitions status to `Rejected` without creating a `Mentorship`. | FR-06, FR-13, NFR-04 |
| `Mentorship` | `end()`, `hasActiveMentorAccess()`; manages active Learner–Mentor pairing, single active Mentorship per pair, access revocation on ending, and history preservation. | FR-05, FR-06, FR-11, FR-13, NFR-04, NFR-05 |
| `MentorshipConversation` | `sendMessage()`; strictly one conversation per Mentorship restricted to its Learner and Human Mentor. | FR-08, FR-11, FR-12, FR-13, NFR-04 |
| `MentorshipMessage` | Ordered message record (`messageOrder`, `content`) within a `MentorshipConversation`. | FR-08, FR-12, NFR-05 |
| `AiMentorConversation` | `presentAiPrivacyNotice()`, `sendAiPrompt()`, `requestAiDataDeletion()`; multiple private Learner AI Mentor conversations separate from Human Mentors; `sendAiPrompt()` requires prior disclosure (via `presentAiPrivacyNotice()` / system disclosure gate before invocation, with no unapproved consent flag or notice entity) of AI data processing, external transmission, and retention/deletion rules. | FR-09, FR-10, FR-14, NFR-02, NFR-03, NFR-04, EIR-02 |
| `AiMentorMessage` | Ordered message record (`senderType`, `messageOrder`, `content`) within an `AiMentorConversation`. | FR-10, NFR-05 |
| `LearningPath` | `recommendRoadmap()`, `regenerateRecommendation()`, `saveRoadmap()`; captures Learner `goal`, `track`, and `currentKnowledge`, regenerates before saving, saves at most one Roadmap per Learning Path, and enforces prior disclosure (via system disclosure gate before invocation) of AI data processing, external transmission, and retention/deletion rules on `recommendRoadmap()` and `regenerateRecommendation()`. | FR-15, FR-16, FR-17, FR-19, FR-20, FR-21, NFR-02, NFR-03, EIR-02 |
| `Roadmap` | `prepareWithAiAssistant()`, `suggestMaterialsWithAiAssistant()`, `addTopic()`, `orderTopics()`, `validateOwnershipXor()`; owned by one Learner, linked exclusively to one `Mentorship` or one `LearningPath`, with AI Assistant preparation keeping Mentor-managed ownership; `suggestMaterialsWithAiAssistant()` invokes `aiService.suggestLearningMaterials(context)` and returns transient resource URL suggestions (`List~string~`) not persisted unless explicitly added as `LearningMaterial` records. **Both AI Assistant methods enforce:** authenticated `mentor` holds `HumanMentor` with `mentor.userId == mentorship.mentorId`; `mentorship.status == Active`; ownership binding (for an existing `Roadmap`, `mentorship.mentorshipId == this.mentorshipId` and `mentorship.learnerId == this.learnerId`; when initializing a new Mentor-managed `Roadmap` via `prepareWithAiAssistant()`, strictly bound to `mentorshipId = mentorship.mentorshipId`, `learnerId = mentorship.learnerId`, and `learningPathId = null`); and prior `NFR-02`/`NFR-03` disclosures to the Mentor and, before any Learner context is sent externally, to the bound Learner (`mentorship.learnerId`) with no notice entity or persisted flag. | FR-07, FR-17, FR-18, FR-20, FR-22, FR-26, FR-27, FR-28, NFR-02, NFR-03, NFR-04, NFR-05, EIR-02; System Design §1 |
| `Topic` | `calculateCompletionRate()`, `isEligibleForTopicAiQuiz()`; ordered learning unit within a Roadmap; triggers optional AI Quiz eligibility at 100% completion on Learner-owned Roadmaps. | FR-22, FR-23, FR-28; System Design §2 |
| `LearningMaterial` | Optional Topic learning resource accessed via `resourceUrl`. | FR-23, FR-24, FR-29 |
| `ExternalService` | Generic supported external service registry for Practical Task verification. | FR-42, FR-43, EIR-03 |
| `LearnerExternalAccount` | `linkExternalAccount()`; links one external account identifier per Learner per `ExternalService`. | FR-42, EIR-03 |
| `PracticalTask` | `verifyViaExternalApi()`, `markCompleted()`; mutual-exclusion completion rules: **External (`isExternal == true`):** `externalServiceId` and `externalActivityIdentifier` are required (non-null); `completionStatus` may transition to `Completed` ONLY after successful `verifyViaExternalApi(...)` using the Roadmap owner's linked `LearnerExternalAccount` (`account.learnerId == Roadmap.learnerId`, derived via `PracticalTask -> Topic -> Roadmap`, and `account.externalServiceId == this.externalServiceId`) against `this.externalActivityIdentifier`; calling `markCompleted(actor)` on an external task MUST be rejected (no manual completion bypass). **Non-external (`isExternal == false`):** `externalServiceId` and `externalActivityIdentifier` must be `null`; `markCompleted(actor)` is permitted ONLY by the responsible active `HumanMentor` (`actor` holds `HumanMentor`, linked `Mentorship.status == Active`, and `actor.userId == Mentorship.mentorId`) on a Mentor-managed Roadmap (`mentorshipId != null`), OR by the owning `Learner` (`actor` holds `Learner` and `actor.userId == Roadmap.learnerId`) on a Learner-owned Roadmap (`learningPathId != null`). Completion status is preserved (`NFR-05`) and kept strictly distinct from Assessment evaluation, Project review, and AI guidance (`NFR-06`). | FR-23, FR-41, FR-42, FR-43, NFR-04, NFR-05, NFR-06, EIR-03 |
| `Assessment` | `submitQuiz()`, `submitTask()`; Quiz or Task Assessment on a Topic (a `Topic` is NOT required to contain an `Assessment`; only when the responsible Mentor adds an `Assessment` to a Mentor-managed `Topic` does that added `Assessment` become required for the Learner per `FR-25`), displaying `taskInstructions` for Task Assessments; both methods enforce `learner.userId == Roadmap.learnerId` and the single-submission-per-Assessment rule (`FR-40`). **Per-question Quiz mapping & instantiation order:** in `submitQuiz()`, both `writtenResponses` (`Map~id, string~`) and `selectedChoiceIdsByQuestion` (`Map~id, id[]~`) are keyed by `quizQuestionId` belonging to this `Assessment` (`writtenResponses` supplies `writtenResponse` for `ShortAnswer` questions; `selectedChoiceIdsByQuestion` supplies `QuizAnswerChoice` IDs validated against each `quizQuestionId` for choice-based questions); `submitQuiz()` enforces `learner.userId == Roadmap.learnerId` and single submission (`FR-40`), creates and persists `AssessmentSubmission` first to generate `assessmentSubmissionId`, and then instantiates and links child `QuizQuestionAnswer` and `QuizSelectedChoice` records. | FR-23, FR-25, FR-30, FR-31, FR-32, FR-36, FR-39, FR-40 |
| `QuizQuestion` | `autoGradeChoice()`, `scoreMultipleSelectAllOrNothing()`; models Single Choice, True/False, Multiple Select (all-or-nothing auto-scoring), and Short Answer (`gradingCriteria` required) questions. | FR-33, FR-34, FR-36, FR-37, FR-38 |
| `QuizAnswerChoice` | Answer option (`choiceText`, `isCorrect`) for choice-based `QuizQuestion` items. | FR-35, FR-36, FR-37 |
| `AssessmentSubmission` | `submitMentorReview()`, `evaluateWithAi()`; single submission per Assessment (`githubUrl` for Task, question answers for Quiz) with at most one review via two mutually exclusive, ownership-enforced paths: for Mentor-managed Roadmaps (`mentorshipId != null`), only `submitMentorReview(mentor, shortAnswerMarks, taskMarks, feedback)` is valid (enforcing `Mentorship.status == Active` and `mentor.userId == Mentorship.mentorId`, storing `reviewerType = HumanMentor` and `reviewerUserId = mentor.userId`); for Learner-owned Roadmaps (`learningPathId != null`), only `evaluateWithAi(aiService)` is valid (enforcing prior `NFR-02`/`NFR-03` disclosure before invocation, calling `aiService.evaluateAssessment(this, outShortAnswerMarks)`, and storing `reviewerType = AiMentor` with `reviewerUserId = null`). **Single-source-of-truth marks & Short Answer propagation:** for `Quiz` submissions (both Mentor and AI paths), `AssessmentReview.marks` is ALWAYS derived strictly as the sum of auto-graded choice `QuizQuestionAnswer.awardedMarks` plus evaluated Short Answer `QuizQuestionAnswer.awardedMarks`—in `submitMentorReview`, `taskMarks` MUST be `null` (not caller-supplied); for Quizzes with `ShortAnswer` questions, both routed evaluators validate each `quizQuestionId` against the submission's `Assessment` (`questionType == ShortAnswer`, `0 <= marks <= QuizQuestion.markAllocation`), write per-question marks to `QuizQuestionAnswer.awardedMarks` via package/internal `~evaluateShortAnswer(marks)` (accessible within the domain package without inheritance and inaccessible to external callers), and persist `AssessmentReview` with that strictly computed sum (in `evaluateWithAi`, never persisting an unverified or conflicting AI-returned aggregate total) and `feedback`. For `Task` submissions (both Mentor and AI paths), `AssessmentReview.marks` stores the supplied submission-level mark (`taskMarks`, required non-null in `submitMentorReview`; or the AI-returned submission-level `marks` from `IExternalAiService.evaluateAssessment` in `evaluateWithAi`). Both `shortAnswerMarks` and `outShortAnswerMarks` are always non-null and MUST be a non-null empty map (`{}`) for `Task` submissions and choice-only `Quiz` submissions (Quizzes with no `ShortAnswer` questions). | FR-30, FR-36, FR-38, FR-40, FR-46, FR-48, FR-49, FR-51, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, EIR-02 |
| `QuizQuestionAnswer` | `~evaluateShortAnswer(marks)`; records Learner's `writtenResponse` (from `writtenResponses` keyed by `quizQuestionId` for `ShortAnswer`) or selected choice links and `awardedMarks` per `QuizQuestion`; instantiated and linked only *after* `Assessment.submitQuiz()` enforces `learner.userId == Roadmap.learnerId` and `FR-40` and persists `AssessmentSubmission` (`assessmentSubmissionId`). `~evaluateShortAnswer(marks)` uses package/internal visibility within the domain package so `AssessmentSubmission`'s routed evaluation methods (`submitMentorReview` and `evaluateWithAi`) can invoke it directly without inheritance while keeping it inaccessible to external callers; it does NOT accept a caller-supplied reviewer type—reviewer authorization and type are derived strictly from the parent submission's `Roadmap` ownership (`AssessmentSubmission -> Assessment -> Topic -> Roadmap`), validating `questionType == ShortAnswer` and `0 <= marks <= QuizQuestion.markAllocation` (for `Task` and choice-only `Quiz` submissions, `shortAnswerMarks` and `outShortAnswerMarks` are always a non-null empty map (`{}`)). | FR-36, FR-37, FR-38, FR-39, FR-40, FR-48 |
| `QuizSelectedChoice` | Maps `QuizAnswerChoice` IDs (supplied via `selectedChoiceIdsByQuestion` keyed by `quizQuestionId` and validated against each `quizQuestionId`) for a choice-based `QuizQuestionAnswer`, instantiated after `Assessment.submitQuiz()` creates `AssessmentSubmission` and `QuizQuestionAnswer`. | FR-36, FR-37, FR-39, FR-40 |
| `AssessmentReview` | `routeReviewer()`; single review (`marks`, `feedback`) visible to the Learner; `reviewerType` and `reviewerUserId` are never caller-selected—`routeReviewer()` resolves ownership so Mentor-managed Roadmaps (`mentorshipId != null`) require `submitMentorReview()` with `Mentorship.status == Active` and `mentor.userId == Mentorship.mentorId` (`reviewerType = HumanMentor`, `reviewerUserId = mentor.userId`), while Learner-owned Roadmaps (`learningPathId != null`) require `evaluateWithAi()` (with prior `NFR-02`/`NFR-03` disclosure) invoking `aiService.evaluateAssessment(this, outShortAnswerMarks)` (`reviewerType = AiMentor`, `reviewerUserId = null`). **Single-source-of-truth `marks`:** for `Quiz` submissions (both Mentor and AI paths), `AssessmentReview.marks` is ALWAYS derived strictly as the sum of auto-graded choice `QuizQuestionAnswer.awardedMarks` plus evaluated Short Answer `QuizQuestionAnswer.awardedMarks` written via `QuizQuestionAnswer.~evaluateShortAnswer(marks)` (`taskMarks` MUST be `null` in `submitMentorReview`, and `evaluateWithAi` strictly computes and persists this derived sum rather than an unverified AI-returned aggregate total); for `Task` submissions (both Mentor and AI paths), `AssessmentReview.marks` stores the supplied submission-level mark (`taskMarks`, required non-null in `submitMentorReview`; or the AI-returned submission-level `marks` from `IExternalAiService.evaluateAssessment` in `evaluateWithAi`). Both `shortAnswerMarks` and `outShortAnswerMarks` are always non-null and MUST be a non-null empty map (`{}`) for `Task` submissions and choice-only `Quiz` submissions. | FR-36, FR-38, FR-46, FR-48, FR-49, FR-51, FR-52, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, EIR-02 |
| `Project` | `submitVersion()`; Topic Project (a `Topic` is NOT required to contain a `Project`; only when the responsible Mentor adds a `Project` to a Mentor-managed `Topic` does that added `Project` become required for the Learner per `FR-25`); before creating each `ProjectSubmission`, `submitVersion(learner, githubUrl)` validates that authenticated `learner` holds `Learner` and `learner.userId == Roadmap.learnerId` (derived via `Project -> Topic -> Roadmap`), creating a new separate `ProjectSubmission` with a monotonically ordered unique `(projectId, submissionNumber)` while preserving all prior submissions and reviews (`FR-45`, `NFR-05`). | FR-23, FR-25, FR-44, FR-45, NFR-04, NFR-05 |
| `ProjectSubmission` | `submitOfficialReview()`, `requestNonOfficialAiGuidance()`; ordered GitHub URL submission (`submissionNumber`, unique per `projectId`) created only when `Project.submitVersion()` validates authenticated `learner` holds `Learner` and `learner.userId == Roadmap.learnerId` (derived via `Project -> Topic -> Roadmap`), preserving all prior submissions and reviews (`FR-45`, `NFR-05`); supports at most one official Mentor review on Mentor-managed Roadmaps and separate non-official AI guidance; `submitOfficialReview()` enforces that the parent `Roadmap` is Mentor-managed (`mentorshipId != null`), its linked `Mentorship.status == Active`, and the authenticated `mentor` holds `HumanMentor` with `mentor.userId == Mentorship.mentorId` (rejecting any other or former Mentor), storing `ProjectReview.mentorId = mentor.userId`; `requestNonOfficialAiGuidance()` enforces prior disclosure (via system disclosure gate before invocation) of AI data processing, external transmission, and retention/deletion rules. | FR-44, FR-45, FR-47, FR-50, FR-52, NFR-02, NFR-03, NFR-04, NFR-05, NFR-06, EIR-02 |
| `ProjectReview` | Official Human Mentor review (`Approved` or `NeedsRevision`, `writtenFeedback`) permitted exclusively when the parent `Roadmap` is Mentor-managed (`mentorshipId != null`), its linked `Mentorship.status == Active`, and the authenticated `mentor` holds `HumanMentor` with `mentor.userId == Mentorship.mentorId` (rejecting any other or former Mentor), storing `mentorId = mentor.userId`; strictly forbidden for Learner-owned Roadmaps and never performed by AI. | FR-47, FR-50, FR-52, NFR-04, NFR-05, NFR-06, EIR-02 |
| `IGoogleAuthProvider` | `authenticateGoogleToken()`; external boundary interface for Google account registration and sign-in. | EIR-01, FR-01, FR-03 |
| `IExternalAiService` | `generateRoadmapRecommendation()`, `suggestLearningMaterials()`, `generateAiResponse()`, `evaluateAssessment()`; external AI service boundary interface (excludes official Project reviews); for Learner-owned `Quiz` submissions containing `ShortAnswer` questions, `evaluateAssessment(submission, outShortAnswerMarks)` populates `outShortAnswerMarks` keyed by `quizQuestionId` (`questionType == ShortAnswer`, `0 <= marks <= QuizQuestion.markAllocation`) so `AssessmentSubmission.evaluateWithAi()` validates and writes each mark via `QuizQuestionAnswer.~evaluateShortAnswer(marks)` and strictly computes and persists `AssessmentReview.marks` as the sum of auto-graded choice marks plus evaluated Short Answer marks (never persisting an unverified or conflicting AI-returned aggregate total); for Learner-owned `Task` submissions, `AssessmentReview.marks` stores the AI-returned submission-level `marks` from `evaluateAssessment(...)`; for `Task` submissions and choice-only `Quiz` submissions (no `ShortAnswer` questions), `outShortAnswerMarks` is always a non-null empty map (`{}`). Every external AI operation (including `evaluateAssessment()`) enforces prior disclosure of AI data processing, external transmission, and retention/deletion rules (`NFR-02`, `NFR-03`) via system disclosure gate before invocation, and when a Human Mentor's AI Assistant request includes Learner data to be transmitted externally, Roadmapy informs the affected Learner *prior to transmission* (`NFR-02`) without adding any notice entity or persisted flag. | EIR-02, FR-10, FR-16, FR-18, FR-36, FR-38, FR-46, FR-47, FR-48, FR-49, FR-51, NFR-02, NFR-03 |
| `IExternalVerificationService` | `verifyTaskCompletion()`; external service API boundary interface for verifying external Practical Task completion. | EIR-03, FR-41, FR-42, FR-43 |
