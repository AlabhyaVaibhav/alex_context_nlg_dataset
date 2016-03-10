
Preparing the data for Crowdflower
----------------------------------

1) Extracting texts from transcribed dialogues:

    alex/alex/tools/reparse/extract_texts.py <call-dir> > src/texts.<id>.tsv

2) Reparsing:

    alex/alex/tools/reparse/reparse_en.py src/texts.<id>.tsv > reparse/reparse.<id>.tsv

3) Abstracting:

    alex/alex/tools/reparse/abstract.sh reparse/reparse.<id>.tsv abstract/abstract.<id>.tsv

4) Generating reply tasks:

    alex/alex/tools/crowdflower/nlg_job/generate_reply_tasks.py abstract/abstract.<id>.tsv > tasks/tasks.<id>.tsv
    (head -n 1 tasks/tasks.<id>.tsv && tail -n +2 tasks/tasks.<id>.tsv | shuf) > tasks/tasks.<id>-shuffled.tsv
