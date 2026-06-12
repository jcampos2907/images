ARG VERSION=2.21.0
FROM n8nio/runners:${VERSION}

# n8n 2.x runs the Code node (JavaScript AND Python "native") inside this
# external task-runner image, NOT in the main n8n container. To use third-party
# Python packages in a Python Code node you must BOTH:
#   1. install them into the python runner's venv (/opt/runners/task-runner-python)
#   2. allowlist them in the launcher config's python "env-overrides"
#      (N8N_RUNNERS_EXTERNAL_ALLOW / N8N_RUNNERS_STDLIB_ALLOW). Setting those as
#      container env vars does NOT work — the launcher overrides them per-runner.
USER root

# markitdown -> convert docs/pdf/pptx/xlsx/html/images/audio to Markdown
# openai     -> OpenAI API client
# Plain markitdown (not markitdown[all]): the base image is Alpine/musl and the
# [all] extra pulls onnxruntime/magika, which have no musllinux wheel.
# markitdown/openai/pandas: doc+sheet inspection for Code nodes.
# Extra engines (imported transitively by pandas/markitdown, so no allowlist
# entry needed): pyxlsb=xlsb, odfpy=ods, python-docx=docx, openpyxl=xlsx/xlsm,
# tabulate=df.to_markdown. xls(xlrd)/pptx/html/images already covered.
RUN cd /opt/runners/task-runner-python \
    && uv pip install --no-cache markitdown openai pandas openpyxl tabulate xlrd pyxlsb odfpy python-docx pymupdf \
    && uv pip show markitdown openai pandas

# Launcher config with markitdown + openai allowlisted for the Python runner.
COPY n8n-task-runners.json /etc/n8n-task-runners.json

USER runner
