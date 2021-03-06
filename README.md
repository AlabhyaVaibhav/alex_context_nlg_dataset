
Alex Context NLG Dataset
========================

* **Authors**: Ondřej Dušek, Filip Jurčíček
* **License**: [Creative Commons 4.0 BY-SA](https://creativecommons.org/licenses/by-sa/4.0/)
* **LINDAT release**: <http://hdl.handle.net/11234/1-1675>
* **Development website**: <https://github.com/UFAL-DSG/alex_context_nlg_dataset>

This is a dataset for NLG in task-oriented spoken dialogue systems, covering the English public
transport information domain. It includes preceding context (user utterance) along with each data
instance (pair of source meaning representation and target natural language paraphrases).
This allows NLG systems trained on this dataset to entrain (adapt) to preceding user utterances,
i.e., reuse wording and syntactic structure. This should presumably improve the perceived
naturalness of the output and may even lead to a higher task success rate. The dataset has been
obtained using crowdsourcing on the CrowdFlower platform.

More information can be found in the following paper:

* Ondřej Dušek and Filip Jurčíček: [*A Context-aware Natural Language Generation Dataset for
    Dialogue Systems*](http://workshop.colips.org/re-wochat/documents/02_Paper_6.pdf). In: 
    RE-WOCHAT, LREC, Portorož 2016.

Please cite the paper if you use this dataset in your research.

The dataset has been used in the following experiments:

* Ondřej Dušek and Filip Jurčíček: [*A Context-aware Natural Language Generator for Dialogue 
    Systems*](http://www.sigdial.org/workshops/conference17/proceedings/pdf/SIGDIAL22.pdf). 
    In: SIGDIAL, Los Angeles 2016.

If you want to compare your results to ours, please look at this paper.


Changes/fixes since the LINDAT release
--------------------------------------
* A bug in delexicalization fixed (“I \*AMPM”) – 1 instance affected
* A discarded paraphrase included – 1 instance affected


Dataset format
--------------

The dataset is released in CSV and JSON formats (`dataset.csv`, `dataset.json`); the contents are
identical. Both files use the UTF-8 encoding.

The dataset contains 1859 instances. Each instance has the following properties:

* `context_utt` -- the context user utterance (the utterance preceding to the response to be
    generated, typically user's question the dialogue system wants to answer). Recorded calls to
    a live dialogue system and manual transcriptions were used to obtain the contexts.
* `context_freq` -- frequency of the context utterance within our recorded calls (each distinct
    context is only included once per response type).
* `context_parse` -- SLU parse of the transcribed context utterance.
* `response_da` -- response semantics (dialogue act) generated by our rule-based bigram policy.
    There may be more responses for the same context; they are stored as separate instances.
* `response_nl` -- response natural language paraphrases. Three paraphrases have been collected on
    CrowdFlower and checked for errors. They are stored as `response_nl1`, `response_nl2`,
    and `response_nl3` in the CSV file.

All properties exist in the default (delexicalized) version and a lexicalized version (with the `-l`
suffix). The lexicalized version was used in CrowdFlower tasks.

The order of the instances is random to allow a simple training/development/test data split.

### The domain ###

The domain is public transport by bus or subway among New York City subway stations
on Manhattan. The users may specify origin and destination stops, departure time, and means of
transport. After a connection is provided, they may ask about its duration, distance, or arrival
time, and number of transfers.

For simplicity, directions provided in the dataset do not involve any transfers.

### Dialogue acts format ###

The dialogue acts in this dataset (`context_parse` and `response_da` properties) follow the [Alex
dialogue act format](https://github.com/UFAL-DSG/alex/blob/master/alex/doc/ufal-dialogue-acts.rst).
Basically, it is a sequence of *dialogue act items*. Each dialogue act item contains an act type
and may also contain a slot and a value.

**Examples**:

| dialogue act                                          | example utterance                   |
| ------------------------------------------------------|-------------------------------------|
| `hello()`                                             | *Hi.*                               |
| `request(from_stop)`                                  | *Where are you coming from?*        |
| `inform(departure_time="7:00")&inform(ampm="pm")`     | *I want departure at 7 pm.*         |
| `iconfirm(to_stop="Central Park")&request(from_stop)` | *OK, from Central Park. Where to?*  |

The **act types used** in this dataset are:
* `inform` -- informing about a connection or a specific detail (distance, arrival time)
* `inform_no_match` -- apology that a route has not been found
* `iconfirm` -- confirmation of user-specified route parameters
* `request` -- request for additional details to complete the route search (origin or destination
    stop)

**Slots used** in this dataset are:
* `from_stop` -- the origin stop
* `to_stop` -- the destination stop
* `direction` -- heading/direction of the bus or subway (used when providing directions)
* `departure_time` -- departure time (absolute, e.g. *7:00*)
* `departure_time_rel` -- departure time (relative, e.g. *in 10 minutes*)
* `ampm` -- daytime (AM/PM, morning or afternoon). This is merged with the departure time when
    giving directions, but separate for confirmations/apologies.
* `vehicle` -- means of transport (bus/subway)
* `line` -- bus or subway line in question (used when providing directions)
* `arrival_time` -- arrival time (absolute; only provided on special request)
* `duration` -- duration of the travel (only provided on special request)
* `distance` -- distance of the travel (only provided on special request)
* `num_transfers` -- number of transfers on the route (only provided on special request)
* `alternative` -- connection variant (next, previous, *n*-th)

### Delexicalization limits ###

Even though the delexicalized versions of all dataset items only contain `*SLOT` placeholders
instead of each slot value, the delexicalization has had its limits. These must be observed to
generate fluent utterances using this dataset:

* The `ampm` slot may contain either the expressions *am*/*pm*, or daytime indication such as
    *morning*, *afternoon* or *evening*. The abbreviated expressions show a different behavior
    than the full words (e.g., *7:00 pm* and *7:00 in the evening* is OK, but *7:00 in the pm*
    is not). This must be checked for when filling the values instead of the placeholder.

* The `ampm` slot is sometimes doubled, as in *7:00 pm in the evening* or *7:00 pm evening*;
    the delexicalization resulted in `*DEPARTURE_TIME *AMPM in the *AMPM` and so on.

* The `num_transfers` slot was used with the values *0*, *1*, *2* in the development of the
    dataset. Each of these values exhibits a slightly different behavior (e.g., singular/plural),
    so a generated sentence might need to be adapted to the concrete value.

* The `alternative` slot has only been delexicalized when its value was numeric (e.g.,
    *second option*). The usage of ordinal numerals is indicated by using the `*ALTERNATIVE-th`
    placeholder, cardinal numerals do not have the `-th` suffix.
    The utterances for `alternative=next`, `alternative=previous`, `alternative=dontcare`,
    and `alternative=last` have not been delexicalized as different expressions can be used and
    underlie entrainment.


Dataset development (technical manual)
--------------------------------------

This documents the development of the dataset and can be adapted for the development of similar
datasets in different domains. A more general description is given in the paper mentioned above.

The [Alex spoken dialogue systems framework](https://github.com/UFAL-DSG/alex) is required for the
development. All scripts relating to the dataset development are located in the
`alex/tools/reparse/` and `alex/tools/crowdflower/nlg_job` subdirectories.

We assume the `devel/` subdirectory as the working directory in all commands. The intermediary
files are stored and versioned in the `devel/` subdirectory in this repository.

### Collecting and transcribing user utterances ###

See [Alex documentation](https://github.com/UFAL-DSG/alex_context_nlg_dataset) for more details.

### Preparing the data for CrowdFlower ###

We assume that in-domain calls have been recorded and transcribed, and are located in `<call-dir>`.

1) Extracting texts from transcribed dialogues:

    alex/tools/reparse/extract_texts.py <call-dir> > src/texts.tsv

2) Reparsing:

    alex/tools/reparse/reparse_en.py src/texts.tsv > reparse/reparse.tsv

3) Delexicalizing:

    alex/tools/reparse/abstract.sh reparse/reparse.tsv abstract/abstract.tsv

4) Generating reply tasks:

    alex/tools/crowdflower/nlg_job/generate_reply_tasks.py -o abstract/abstract.tsv > tasks/tasks.tsv
    (head -n 1 tasks/tasks.tsv && tail -n +2 tasks/tasks.tsv | shuf) > tasks/tasks-shuffled.tsv

   The reply tasks are shuffled so that they do not appear in a regular order on CrowdFlower
   (otherwise, CF users would always get very similar tasks in one batch).
   The `-o` switch stores context frequency information with each task.

### Running the CrowdFlower jobs ###

All the required files for the CF task are located in `alex/tools/crowdflower/nlg_job`.  Create a
new empty CF job (*New job* -> *See more...* -> *Start from scratch*), then use the files provided
on the *Design* pane in the following way:

* Copy `CFJOB-reply.html` into the *CML* field (turn off WYSIWYG first)
* Click on *Show custom CSS/JS* and add `CFJOB-reply.js` into the Javascript field and
  `CFJOB-reply.css` into the CSS field
* Copy `CFJOB-reply.instructions.html` into the instructions field (turn off WYSIWYG first)

On the *Data* pane, you can upload the data you created in the data preparation step.

The CF task must be supplemented by a spellchecking server. The server must run on a server that
has a SSL certificate, and must have access to the certificate (see `server_langid.py` for details).
CF tasks will not accept a non-SSL connection. Run the server and change the server URL in the CF
job's Javascript field accordingly (the `serverURL` variable).

### Checking CrowdFlower results ###

Since the checks implemented in the Javascript code are not and cannot be 100% accurate, results
obtained from CF need to be checked manually for errors. This is done using the `checker.py`
script:

    alex/tools/crowdflower/nlg_job/checker.py tasks/tasks.tsv postprocessed.csv \
        cf_results.csv [cf_results2.csv ...]

You will then interactively accept or reject each CF user's response (Y/N) and optionally also
assess  whether it contains lexical and/or syntactic entrainment (L/S, separated by space). You
may also postedit responses (just append the postedit text after your accepting judgment, separated
by two spaces).

After your check is done, you might want to resubmit the unsuccessful replies to CF in order to get
better replies. You can assemble all the unfinished tasks using this command:

    alex/tools/crowdflower/nlg_job/get_unfinished.py tasks/tasks-shuffled.tsv postprocessed.csv

This will print all unfinished tasks to standard output, with comments specifying how many CF
judgments are still required (out of the preset 3 judgments).

### Assembling the dataset ###

The final dataset assembly does further normalization and checking. You will be presented with
all spellchecking errors (CF users are allowed 1 error per 10 tokens) and unexpected final
punctuation; otherwise, the process is automatic. To assemble the dataset, run the following
command:

    alex/tools/crowdflower/nlg_job/build_dataset.py tasks.tsv <dataset_name> postprocessed.csv

This will save the files `<dataset_name>.csv` and `<dataset_name>.json`. The data instances are
shuffled on the output.


Acknowledgments
---------------

This work was funded by the Ministry of Education, Youth and Sports of the Czech Republic under
the grant agreement LK11221 and core research funding, SVV project 260 333, and GAUK grant 2058214
of Charles University in Prague. It used language resources stored and distributed by the
LINDAT/CLARIN project of the Ministry of Education, Youth and Sports of the Czech Republic
(project LM2015071).

