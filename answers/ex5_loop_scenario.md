# Ex5 — Edinburgh research loop scenario

## Your answer

In the successful real-mode run, session `sess_918142e47522`, the Ex5
scenario produced two loop-half subgoals: sg_1 researched the Haymarket
booking facts, and sg_2 wrote the HTML flyer and completed the task.

Turn 1 called venue_search, get_weather, and calculate_cost in parallel
— all three are parallel_safe because they only read fixtures. Turn 2
wrote the flyer via generate_flyer, which is parallel_safe=False
because it writes a file. The first generate_flyer attempt was missing
event_details, but the executor recovered by re-calling the read tools,
then calling generate_flyer with the full venue, weather, total, and
deposit facts.

The dataflow integrity check then verified 4 concrete flyer facts
against tool outputs. This is the critical guard: the flyer is a
polished artifact, but it is only trustworthy if the money, weather
condition, and temperature came from tools rather than model invention.

## Citations

- Real run `make ex5-real`, session `sess_918142e47522` — produced `workspace/flyer.html`
- `make narrate SESSION=sess_918142e47522` — tool sequence and recovery after failed flyer call
- starter/edinburgh_research/run.py — scripted tool sequence and integrity check
- starter/edinburgh_research/tools.py — implemented tool outputs used by the flyer
