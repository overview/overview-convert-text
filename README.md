Converts text files for [Overview](https://github.com/overview/overview-server).

# Methodology

This program always outputs `0.json`, `0.txt` and `0.blob` and
`0-thumbnail.jpg`.

The output JSON has `"wantOcr": false` and `"contentType": "application/pdf".
Other input JSON (in particular, `"wantSplitByPage"`) is passed through.

We guess character set using [chardet](https://github.com/chardet/chardet).
If the user wants to guarantee a certain character set, the user must encode
the text as UTF-8: it's the only character set we detect 100% accurately.

We generate the PDF using [ReportLab](https://www.reportlab.com/). It's the
fastest, despite its overpowered API.

We generate a thumbnail using `pdftocairo`. We output JPG: it saves ~0.03s.

Right now, we only embed Noto Sans Mono. The annoying reality is that TTF
fonts support a maximum of 64k characters. TODO solve this problem, so we
can display text in all supported languages.

# Testing

Write to `test/test-XYZ`. `docker build .` will run the tests.

Each test has `input.blob` (which means the same as in production) and
`input.json` (whose contents are `$1` in `do-convert-single-file`). The files
`stdout`, `0.json`, `0.blob`, `0.txt`, and `0-thumbnail.(png|jpg)` in the
test directory are expected values. If actual values differ from expected
values, the test fails.

PDF, PNG and JPEG are tricky formats to get exactly right. You may need to use
the Docker image itself to generate expected output files. For instance, this is
how we built `test/test-1page/0-thumbnail.png`:

1. Wrote `test/test-1page/{input.json,input.blob,0.txt,0.blob,stdout}`
1. Ran `docker build .`. The end of the output looked like this:
    ```
    Step 12/13 : RUN [ "/app/test-convert-single-file" ]
     ---> Running in f65521f3a30c
    1..3
    not ok 1 - test-1page
        do-convert-single-file wrote /tmp/test-do-convert-single-file912093989/0-thumbnail.jpg, but we expected it not to exist
    ...
    ```
1. `docker cp f65521f3a30c:/tmp/test-do-convert-single-file912093989/0-thumbnail.jpg test/test-1page/`
1. `docker rm -f f65521f3a30c`
