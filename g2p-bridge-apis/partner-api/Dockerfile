FROM python:3.12.11-alpine3.22

ARG G2P_REF=v2.1.0

RUN apk add --no-cache --virtual .build-deps gcc libc-dev linux-headers make \
 && apk add --no-cache bash git gettext libpq-dev libmagic

WORKDIR /app

# Core libs (as in your current API Dockerfile)
RUN pip install --no-cache-dir "git+https://github.com/psnappz/openg2p-fastapi-common@1.1#subdirectory=openg2p-fastapi-common" \
    && pip install --no-cache-dir "git+https://github.com/psnappz/openg2p-fastapi-common@1.1#subdirectory=openg2p-fastapi-auth-models" \
    && pip install --no-cache-dir "git+https://github.com/psnappz/openg2p-fastapi-common@1.1#subdirectory=openg2p-fastapi-auth" \
    && pip install --no-cache-dir "git+https://github.com/psnappz/openg2p-fastapi-common@1.1#subdirectory=openg2p-fastapi-partner-auth"

# Always from openg2p-g2p-bridge repo (no local installs)
RUN pip install --no-cache-dir \
    "git+https://github.com/OpenG2P/openg2p-g2p-bridge@${G2P_REF}#subdirectory=openg2p-g2p-bridge-models" \
    "git+https://github.com/OpenG2P/openg2p-g2p-bridge@${G2P_REF}#subdirectory=openg2p-g2p-bridge-partner-api"

# Dynamic deps from workflow (optional)
COPY adapters.requirements.txt /tmp/adapters.requirements.txt
RUN if [ -s /tmp/adapters.requirements.txt ]; then pip install --no-cache-dir -r /tmp/adapters.requirements.txt; fi

RUN apk del --no-network .build-deps

ENV PYTHONUNBUFFERED=1
ENV G2P_BRIDGE_NO_OF_WORKERS=2
ENV G2P_BRIDGE_HOST=0.0.0.0
ENV G2P_BRIDGE_PORT=8000
ENV G2P_BRIDGE_WORKER_TYPE=gunicorn

CMD python3 -m openg2p_g2p_bridge_partner_api.main migrate; \
    gunicorn "openg2p_g2p_bridge_partner_api.main:app" --workers ${G2P_BRIDGE_NO_OF_WORKERS} --worker-class uvicorn.workers.UvicornWorker --bind ${G2P_BRIDGE_HOST}:${G2P_BRIDGE_PORT} --log-level info
