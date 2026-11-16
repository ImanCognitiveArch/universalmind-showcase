# UniversalMind Evaluation SDK â€” 30-Minute Evaluation Guide

**Version:** 0.3.0a1-eval
**Status:** EVALUATION ONLY â€” Not for production use
**Audience:** First-time external technical evaluator
**Time required:** 30 minutes, no prior knowledge of the codebase
**Network:** Local only â€” do not expose to 0.0.0.0 or a public network

---

## 1. What This Is / What This Is Not

**This IS:**
- A human-governed Decision Review Workbench
- A way to submit a decision, see a recommendation, and keep the final decision with a human
- A clean, installable SDK that runs locally from a wheel

**This IS NOT:**
- General intelligence
- An autonomous agent that takes action by itself
- A system for high-stakes, medical, legal, or safety-critical decisions
- A production deployment. There is no authentication, no persistence guarantee, and no hosted service in this version.

If at any point the product seems to claim otherwise, that is a bug in documentation or UI. Please report it.

## 2. Ground Rules for This Evaluation

1. **You are the authority.** The system recommends. You decide.
2. **Evaluation input stays local at runtime.** This build has no configured outbound model/API dependency or product telemetry. Package installation may of course contact your configured Python package index.
3. **Do not try to â€œmake it work.â€** If something is confusing, slow, or unclear, stop and write it down. Your friction is the deliverable.
4. **Do not invent results.** Only report what you actually saw on screen / in the API response.
5. **You do not need to understand the architecture.** If the product only makes sense after an architecture lecture, that is a failure of the product, not of you.

## 3. Prerequisites

- Python 3.10+ (3.11 recommended)
- A fresh virtual environment is recommended
- The wheel file we provided: `universalmind_eval-0.3.0a1-py3-none-any.whl`
- A browser (for the Workbench UI)
- No API keys and no product account

## 4. The 30-Minute Plan

| Minutes | What you do | Goal |
| :--- | :--- | :--- |
| 0â€“5 | Read Section 1 and 5 | Understand what you are evaluating |
| 5â€“10 | Install + `doctor` | Prove clean install |
| 10â€“20 | Run Scenario A | See a full review lifecycle |
| 20â€“25 | Make a human decision | Test the governance boundary |
| 25â€“30 | Fill the Friction Log | Tell us what broke / confused you |

Set a timer. If you get stuck for more than 5 minutes on any step, stop and log it. That is a valid evaluation result.

---

## 5. What UniversalMind Does in One Paragraph

You bring a decision with trade-offs. UniversalMind returns a structured review: a recommendation, alternatives, confidence, decision quality, and governance state. It then waits. It does not dispatch the proposed governed action until you explicitly approve; rejection records a governed rejection without dispatch. Your human decision is tied to the same product `review_id` and governance `action_id`. Your final observation about what actually happened can then be recorded against that same review/action pair. That audit trail is the product.

Key terms you will see:

- **review_id:** Process-lifetime identity of this review. Use this exact ID for all follow-up calls.
- **action_id:** Governance identity of the proposed action. It must not change across the human decision lifecycle.
- **Recommendation:** What the system suggests. Not a decision.
- **EMA eligibility:** Whether the action is *eligible* for a certain governance path. Eligibility is NOT approval.
- **pending_human:** The state where the system is waiting for you. This is the correct stopping point, not an error.

## 6. Step-by-Step

### Step 1 â€” Install (5 minutes)

```bash
# In a fresh venv
pip install universalmind_eval-0.3.0a1-py3-none-any.whl

universalmind info
universalmind doctor
```

**Expected:**
```text
[PASS] python
[PASS] product_api
[PASS] static_ui
UniversalMind doctor: PASS
```

**If `doctor` fails:** Do not try to fix it. Copy the full output into the Friction Log. That is your result.

Check that imports come from `site-packages`, not from a source checkout:

```bash
python -c "import universalmind; print(universalmind.__file__)"
```

### Step 2 â€” Start the Workbench (2 minutes)

```bash
universalmind serve
```

- It must bind to `127.0.0.1` only by default.
- The CLI must refuse a non-loopback host such as `0.0.0.0` in this version.
- Open `http://127.0.0.1:8000` in your browser.

Smoke test:
- `GET /health` â†’ `status = ok`
- `GET /` â†’ redirects to `/static/index.html` (HTTP 307)

### Step 3 â€” First-User Comprehension (3 minutes, do not run anything yet)

Before you click RUN, answer for yourself:

1. Can you tell what UniversalMind does from the UI alone?
2. Can you tell what RUN will do?
3. Can you tell that a recommendation is not a decision?
4. Can you tell when *your* authority begins?

If any answer is â€œnoâ€ or â€œnot sureâ€, write it down. Do not read source code to figure it out.

### Step 4 â€” Run Scenario A (10 minutes)

Use this exact scenario. Do not change the wording for your first run:

> **Scenario A:** â€œI need to choose between launching a project quickly at lower cost with higher execution risk, or launching more slowly at higher cost with lower execution risk.â€

Submit it through the Workbench UI / product API path.

Capture and save:
- `review_id`
- The proposed `action_id` from the governance/action data
- Recommendation
- Confidence
- Alternatives listed
- Decision quality
- EMA eligibility
- Governance state (should be `pending_human`)

**Stop here.** Do not approve automatically. The system should be waiting for you.

### Step 5 â€” Human Decision Boundary (5 minutes)

You are now being asked to decide. The UI should make it clear:

- What you are being asked to approve or reject
- That the proposal does not dispatch until you explicitly approve
- That rejection is recorded without dispatch
- That `action_id` stays the same

Choose one, using the *exact* `review_id` from Step 4:

**Option A â€” Approve:**
```text
POST /approve  { "review_id": "<exact review_id from Step 4>" }
```

**Option B â€” Reject:**
```text
POST /reject  { "review_id": "<exact review_id from Step 4>" }
```

Verify after:
- The governance state matches your choice
- The response receipt `action_id` matches the proposed `action_id` from Step 4
- Approval dispatch occurred only after your explicit approval; rejection did not dispatch
- Keep using the same original `review_id` for the outcome step

### Step 6 â€” Human Outcome (3 minutes)

Now tell the system what you actually observed / chose in the real world. Use your own words. Do not let the system invent it.

```text
POST /outcome  { "review_id": "<same review_id>", "summary": "<your brief observation>" }
```

Verify:
- The outcome response contains the same `review_id`
- The outcome `action_id` matches the governed action
- No new governed action or dispatch was created by recording the observation

### Step 7 â€” Stop the Server

Stop `universalmind serve`. No persistence is required for this evaluation to count.

---

## 7. What Success Looks Like

This evaluation is successful if you can answer YES to these:

- [ ] I installed from the wheel without using source code
- [ ] `doctor` passed, or I captured why it failed
- [ ] I ran Scenario A and got a `review_id` + proposed `action_id`
- [ ] I understood that I had to decide, and the system waited for me
- [ ] My approve/reject receipt preserved the same `action_id`
- [ ] My outcome was recorded against the same review/action pair without creating a new action

It is NOT required that you agree with the recommendation. Disagreement is valid data.

## 8. Known Limitations â€” Please Do Not Work Around Them

These are intentional limits in 0.3.0a1-eval:

- **No authentication.** That is why it is local-only.
- **No production use.** Do not deploy this.
- **No persistence guarantee.** Do not rely on it as a system of record.
- **SCL / Stigmergic memory is on RUNTIME HOLD.** A recent preregistered experiment (SCL-1.3A, 816 triple-failure scenarios) did NOT pass all success criteria. Simple TTL / linear-decay hypotheses were rejected. We are not shipping stale-memory handling as a production capability. The research artifact is quarantined and not merged into this product path.
- **No claims about robotics, spacecraft, or silicon deployment.** This SDK has not been deployed on Atlas, Spot, spacecraft, or silicon.

If the UI or docs imply otherwise, that is a documentation bug. Report it.

## 9. Friction Log â€” This Is Your Main Deliverable

Copy this, fill it, and send it back. One line per friction is enough. We prefer blunt over polite.

```text
Evaluator: [name / role, optional]
Date: [YYYY-MM-DD]
Environment: [OS, Python version]
Time spent: [minutes]

1. First confusion (minute __):
   What I expected:
   What I saw:
   What I did:

2. Biggest obstacle to finishing:
   ...

3. One thing I would change if I owned this product:
   ...

4. Would I run this again for a second scenario? [Yes / No / Only if ___]
   Why:

5. In one sentence, what does this product do?
   ...

6. In one sentence, what does it NOT do?
   ...
```

Also answer these 7 quick questions (Yes / No / Unsure):

A. Could a non-engineer understand the product purpose from the UI?
B. Could I understand the evidence shown?
C. Could I distinguish recommendation from decision?
D. Could I understand EMA eligibility?
E. Could I understand when my authority begins?
F. Could I understand the governance receipt (`review_id` / `action_id`)?
G. Did the product demonstrate value without requiring an architecture explanation?

## 10. How to Send Feedback

Send the Friction Log + the `review_id` / `action_id` from your run + any `doctor` output if it failed.

No need to write a formal report. The log above is enough.

---

## Appendix â€” CLI Reference

```bash
universalmind info    # Show version and evaluation boundary
universalmind doctor  # Check python / product_api / static_ui
universalmind serve   # Start local Workbench on 127.0.0.1 only
```

## Appendix â€” Endpoints Used in This Guide

- `GET /health`
- `GET /` â†’ `307` â†’ `/static/index.html`
- `POST /approve`
- `POST /reject`
- `POST /outcome`

---

**End of Guide**

Thank you for evaluating instead of assuming. Your friction is the backlog.
