# syntax=docker/dockerfile:1.14
ARG IGNITION_VERSION
FROM inductiveautomation/ignition:${IGNITION_VERSION:-latest} AS prep

# Temporarily become root for system-level updates (required for 8.1.26+)
USER root

# Install some prerequisite packages
RUN apt-get update && apt-get install -y wget ca-certificates jq zip unzip sqlite3

# Set working directory for this prep image and ensure that exits from sub-shells bubble up and report an error
WORKDIR /root
SHELL [ "/usr/bin/env", "-S", "bash", "-euo", "pipefail", "-c" ]

# Copy the module files to the container
COPY --chmod=0644 modules/*.modl ./

# Set CERTIFICATES/EULAS acceptance in gateway backup config db
COPY base.gwbk .
COPY --chmod=0755 register-module.sh register-password.sh ./

# Modify register-module.sh to be more robust
RUN sed -i 's/set -euo pipefail/set -uo pipefail/' register-module.sh && \
    sed -i 's/shopt -s inherit_errexit//' register-module.sh

ARG GATEWAY_ADMIN_USERNAME="admin"
RUN --mount=type=secret,id=gateway-admin-password \
    unzip -q base.gwbk db_backup_sqlite.idb && \
    shopt -s nullglob; \
    for module in *.modl; do \
      echo "Processing module: ${module}" && \
      ./register-module.sh \
        -f "${module}" \
        -d db_backup_sqlite.idb || echo "WARNING: Failed to register module ${module}, continuing..."; \
    done; \
    shopt -u nullglob && \
    ./register-password.sh \
      -u "${GATEWAY_ADMIN_USERNAME}" \
      -f /run/secrets/gateway-admin-password \
      -d db_backup_sqlite.idb && \
    zip -q -f base.gwbk db_backup_sqlite.idb || \
    if [[ ${ZIP_EXIT_CODE:=$?} == 12 ]]; then \
      echo "No changes to internal database needed during module registration."; \
    else \
      echo "WARNING: Error (${ZIP_EXIT_CODE}) encountered during re-packaging of config db."; \
    fi

# Final Image
FROM inductiveautomation/ignition:${IGNITION_VERSION:-latest} AS final

# Temporarily become root for system-level updates (required for 8.1.26+)
USER root

# Add supplemental packages, such as git if needed/desired
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    git && \
    rm -rf /var/lib/apt/lists/*

# Copy files from the prep stage to the final image
COPY --from=prep --chown=ignition:ignition /root/*.modl ${IGNITION_INSTALL_LOCATION}/user-lib/modules/
COPY --from=prep --chown=ignition:ignition /root/base.gwbk ${IGNITION_INSTALL_LOCATION}/base.gwbk
COPY --chmod=0755 --chown=root:root docker-entrypoint-shim.sh /usr/local/bin/

# Return to ignition user
USER ignition

# Supplement other default environment variables
ENV ACCEPT_IGNITION_EULA=Y \
    IGNITION_EDITION=standard \
    GATEWAY_MODULES_ENABLED=all

# Target the entrypoint shim for any custom logic prior to gateway launch
ENTRYPOINT [ "docker-entrypoint-shim.sh" ]