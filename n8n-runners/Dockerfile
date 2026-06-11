ARG VERSION=2.21.0
FROM n8nio/runners:${VERSION}

# n8n 2.x runs the Code node (JavaScript AND Python "native") inside this
# external task-runner image, NOT in the main n8n container. Third-party
# Python packages must therefore be installed into the python task runner's
# own venv (/opt/runners/task-runner-python) and then allowlisted at runtime
# via N8N_RUNNERS_EXTERNAL_ALLOW (set on the runner container in k8s).
USER root

# markitdown[all] -> convert docs/pdf/pptx/xlsx/html/images to Markdown
# openai           -> OpenAI API client
RUN cd /opt/runners/task-runner-python \
    && uv pip install --no-cache 'markitdown[all]' openai \
    && uv pip show markitdown openai

USER runner
