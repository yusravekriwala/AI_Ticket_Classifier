# AI Support Ticket Triage

**Support queues are usually worked in the order tickets arrive. That means a customer whose account is locked waits behind three people asking about delivery dates.**

This project reads an incoming support ticket and assigns it a priority — low, medium or high — before a human touches it, so agents open the urgent ones first. It works in English and German, gives a plain-language reason for every decision, and costs a fraction of a cent per ticket.

Built as a working prototype on a public dataset of 28,587 real support tickets.

---

## What it does, in one example

A ticket arrives:

> *Dear Support, since 09:00 this morning nobody in our Milan office can log in to the platform. About 40 people are affected and we cannot process orders.*

The classifier returns:

```json
{
  "priority": "high",
  "confidence": 95,
  "reasoning": "A core service is unavailable and roughly 40 users are blocked from working, with direct revenue impact.",
  "key_factors": ["service unavailable", "many users affected", "revenue impact"]
}
```

The agent sees the priority **and the reasoning**. That second part matters more than it looks. A tool that says *"high, trust me"* is one nobody trusts. A tool that shows its working is one people will argue with, correct, and eventually rely on.

---

## The finding that shaped this project

My first prompt defined priority the way most people would: *security issues are high, data loss is high, urgent language is high.* It scored poorly on the first pilot run — and the errors were not random. **Every misclassification was the model escalating above the human label, never below it.**

Looking at the disagreements, the pattern was obvious. This ticket is labelled **low** priority by a human:

> *Sehr geehrter Kundenservice, ich schreibe, um Rat zur Sicherung medizinischer Daten innerhalb einer WordPress-Website einzuholen…*
>
> ("I'm writing to ask for advice on securing medical data on a WordPress site.")

Medical data. Security. Compliance. My prompt saw those words and returned **high**. But nothing is broken — the customer is *asking a question*. It can wait until Thursday.

The real rule in the data is not *how serious does the topic sound*, it is **is something failing right now?** A request about a frightening subject is low priority. An incident about a boring subject is not. Notebook 01 confirms this holds across the dataset: `Request` tickets skew low regardless of subject matter, `Incident` tickets skew high.

Prompt v2 makes that the first question the model asks, before it looks at subject matter at all, and adds calibration examples of exactly this trap. Both prompts are kept in [`src/classifier.py`](src/classifier.py) so the before/after comparison is reproducible rather than a claim.

**The transferable lesson:** the failure was not the model's. It was mine, for writing a prompt from my assumptions about the business instead of from the business's actual decisions. Getting that right is a conversation with the support team, not a coding problem.

---

## How it's evaluated

[`notebooks/03_evaluation.ipynb`](notebooks/03_evaluation.ipynb) runs a stratified sample of 180 tickets — 60 from each priority level — through both prompt versions and reports:

- **Accuracy overall and per priority level**, against a majority-class baseline. The raw data is 40% medium, so anything below ~40% is worse than guessing.
- **Direction of error** — over-escalation versus under-escalation. These are not the same failure. Marking everything urgent floods the priority queue and the team stops looking at it within a week; missing a real incident means a customer sits unattended. A tool at 80% accuracy that fails safe is more useful than one at 85% that fails randomly.
- **Confusion matrices** for both prompt versions side by side.
- **Accuracy by language and by ticket type**, to catch the case where the model works in English and quietly fails in German.
- **Confidence calibration** — whether accuracy actually rises with the model's stated confidence. If it does, the score can be used as an auto-file threshold. If it doesn't, it's decoration and shouldn't be shown to agents.
- **Cost and median latency per ticket**, projected to monthly volume — because "is this worth deploying" is a budget question, not an accuracy question.

Results are cached to `results/` so re-running the notebook doesn't re-spend API calls, and the sample is stratified deliberately: the raw data is only 21% low-priority, and low is the class the model finds hardest. An unstratified sample would hide that.

---

## Try it

You need Python 3.10+ and a free [Groq API key](https://console.groq.com).

```bash
git clone <this repository>
cd ai-support-ticket-triage

python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate

pip install -r requirements.txt

cp .env.example .env             # Windows: copy .env.example .env
# open .env and paste your API key
```

Download the dataset (link below), save it as `data/tickets.csv`, then open the notebooks in order.

```python
from classifier import TicketClassifier

clf = TicketClassifier(provider="groq")
result = clf.classify(
    subject="Cannot access account",
    body="Nobody in our office can log in since this morning.",
    language="en",
)
print(result.priority, result.confidence, result.reasoning)
```

---

## What's in here

| File | What it is |
|---|---|
| [`notebooks/01_data_exploration.ipynb`](notebooks/01_data_exploration.ipynb) | What the tickets look like, how priorities are distributed, and the data finding that drove the prompt rewrite |
| [`notebooks/02_classifier_demo.ipynb`](notebooks/02_classifier_demo.ipynb) | The classifier running on individual tickets, including the v1-vs-v2 failure case |
| [`notebooks/03_evaluation.ipynb`](notebooks/03_evaluation.ipynb) | The measurement: accuracy, confusion matrices, error direction, cost, latency |
| [`src/classifier.py`](src/classifier.py) | The actual logic — importable, retries on failure, runs against Groq or Claude |
| [`docs/agent_playbook.md`](docs/agent_playbook.md) | One page for the support agent who has to use this. No jargon |
| [`docs/rollout_plan.md`](docs/rollout_plan.md) | How I'd introduce it to a team without them ignoring it |
| [`automation/n8n_ticket_triage.json`](automation/n8n_ticket_triage.json) | An n8n workflow that wires it to a live inbox |

**Dataset:** [Multilingual Customer Support Tickets](https://www.kaggle.com/datasets/tobiasbueck/multilingual-customer-support-tickets) — 28,587 tickets, German and English, 10 queues, human priority labels. Not committed here; download it and drop it in `data/`.

---

## Design decisions worth explaining

**Why an LLM rather than a trained classifier?** With 28,000 labelled rows, a fine-tuned model would likely be more accurate and much cheaper per call. But a support team's definition of "urgent" changes — a product launches, a compliance deadline lands, a big customer escalates. Changing a prompt takes ten minutes and no retraining, and a non-technical team lead can read it and tell you it's wrong. That maintainability is worth more than a few accuracy points in a tool whose real job is to be trusted and adjusted by the people using it.

**Why the model never sees `type` or `queue`.** Those columns exist in the dataset and correlate strongly with priority — but at the moment a ticket actually arrives, nobody has filled them in. Feeding them to the model would leak the answer and produce a number that collapses in production.

**Why the provider is swappable.** `TicketClassifier(provider="groq")` and `provider="anthropic"` run the same prompt through the same evaluation harness. Prototyping against a free tier and deploying against whatever the company already pays for shouldn't require a rewrite.

**Why the reasoning field exists.** Adoption, not accuracy. See the playbook.

---

## Known gaps and what I'd do next

This is a prototype. Being specific about what's missing is more useful than pretending it isn't.

| Gap | Why it matters | What I'd do |
|---|---|---|
| **Evaluated on hundreds of tickets, not thousands** | Per-class accuracy on 60 samples has a wide confidence interval — a 5-point gap between prompts may be noise | Run the full 28k set in batches, report confidence intervals, bootstrap the prompt comparison |
| **No human agreement baseline** | I measure the model against dataset labels, but I don't know how often two experienced agents agree with each other. If humans agree 80% of the time, an 80% model is already at ceiling | Have 2–3 agents independently label 100 tickets, measure inter-rater agreement, use that as the real target |
| **Some dataset labels are wrong** | A reported medical-data breach carries a `low` label. Measuring against bad labels understates real performance | Agree a written labelling rubric with the support team, relabel a gold set against it, measure on that |
| **Routes by priority, not by queue** | The dataset has 10 queues. Priority answers *when*, not *who* | Extend the same prompt structure to predict queue — a harder 10-class problem, evaluated separately |
| **No cost ceiling or rate-limit handling at volume** | Retries with backoff exist, but sustained throughput isn't tested | Batch endpoint, concurrency control, a daily spend cap, and a keyword-rule fallback if the API is down |
| **Confidence score is not yet calibrated** | The model says "95%" but that number has no statistical meaning until it's checked against outcomes | Notebook 03 tests whether accuracy rises with confidence. If it does, use it as an auto-file threshold; if not, hide it from agents |
| **No feedback loop** | Every agent override is training signal, and right now it's discarded | Log overrides, review disagreements monthly, feed recurring patterns back into the prompt as new calibration examples |
| **Not tested on the languages that matter** | This dataset is German and English. A real multi-site deployment would need Italian, Spanish and Dutch | Translate a labelled sample, measure per-language accuracy, add calibration examples where it drops |

The two I'd do first are the **human agreement baseline** and the **feedback loop**. Both are about making sure the thing is aimed at the right target, which is worth more than another two accuracy points against a target that might be wrong.

---

## License

MIT — see [LICENSE](LICENSE).
