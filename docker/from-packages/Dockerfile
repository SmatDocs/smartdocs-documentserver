# syntax=docker/dockerfile:1.4
#
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

FROM debian:stable-slim

# some descriptive metadata - docker image inspect
ARG author
LABEL author=${author:-"Collabora Productivity Ltd."}
LABEL description="Collabora Online is a powerful collaborative Office suite that supports all major document, spreadsheet and presentation file formats, which you can integrate into your own infrastructure. Collabora Online provides data security and sovereignty, and is ideally suited to the demands of a modern distributed working environment. Delivering a familiar look and feel, Collabora Online represents a real alternative to other big-brands solutions, giving you control and flexibility."
ARG releasenotes
LABEL release.notes=${releasenotes:-"https://www.collaboraonline.com/code-24-04-release-notes/"}
ARG version
LABEL version=${version:-"24.04.1.3"}
ARG coretag
LABEL commit.history.core="https://git.libreoffice.org/core/+log/${coretag}"
ARG onlinetag
LABEL commit.history.online="https://github.com/CollaboraOnline/online/commits/${onlinetag}"

# repo: can be 'repos', 'repos-staging', 'repos-snapshot'
# Optional. The default is 'repos'.
ARG repo

# type: can be
# 'code' - Collabora Online Development Edition
# 'cool' - Collabora Online, to build this you need to give your secret URL part from https://support.collaboraoffice.com, i.e. you have to be Collabora partner or customer
# 'key'  - Collabora Online, the license key enabled version.
# Optional. The default is 'code'.
ARG type

# Optional. If defined then brand package will not be installed.
ARG nobrand

# UTF-8 locale is needed to handle file names with non-ASCII characters
ENV LC_CTYPE C.UTF-8

# Mount secret URL part (for type 'cool') securely
RUN --mount=type=secret,id=secret_key \
# Update repos before installing packages
    apt-get update && \
# Install dependencies of installer of Collabora Online
    apt-get -y install cpio tzdata libcap2-bin apt-transport-https gnupg2 ca-certificates curl && \
# Setup Collabora repo
    repourl="https://collaboraoffice.com/${repo:-repos}/CollaboraOnline/"; \
    secret_key=$(cat /run/secrets/secret_key); \
    if [ "$type" = "cool" ] && [ -n ${secret_key+set} ]; then \
        echo "Based on the provided build arguments Collabora Online from customer repo will be used."; \
        repourl="${repourl}24.04/customer-deb-${secret_key}/"; \
    elif [ "$type" = "key" ]; then \
        echo "Based on the provided build arguments license key enabled Collabora Online will be used."; \
        repourl="${repourl}24.04-key/"; \
    else \
        echo "Based on the provided build arguments Collabora Online Development Edition will be used."; \
        repourl="${repourl}24.04-CODE/CODE-deb/"; \
    fi && \
    echo "deb [signed-by=/usr/share/keyrings/collaboraonline-release-keyring.gpg] ${repourl} /" > /etc/apt/sources.list.d/collabora.list && \
# Download Collabora package signing key
    if [ "$repo" = "repos-snapshot" ]; then \
        curl https://www.collaboraoffice.com/downloads/gpg/collaboraonline-snapshot-keyring.gpg --output /usr/share/keyrings/collaboraonline-snapshot-keyring.gpg; \
        sed -i "s/collaboraonline-release-keyring/collaboraonline-snapshot-keyring/" /etc/apt/sources.list.d/collabora.list; \
    else \
        curl https://www.collaboraoffice.com/downloads/gpg/collaboraonline-release-keyring.gpg --output /usr/share/keyrings/collaboraonline-release-keyring.gpg; \
    fi && \
# Update repos again
    apt-get update && \
# Install Collabora Online (Collabora Office is a dependency)
# --- ADDED collaboraofficebasis-impress HERE ---
    apt-get -y install coolwsd coolwsd-deprecated \
                       collaboraofficebasis-impress \

# For working with python scripts invoked using UNO command
                       collaboraofficebasis-python-script-provider \
                       collaboraofficebasis-pyuno \
# Install all dictionaries
                       collaboraoffice-dict-* \
# Install all language packs
                       collaboraofficebasis-ar \
                       collaboraofficebasis-bg \
                       collaboraofficebasis-ca \
                       collaboraofficebasis-cs \
                       collaboraofficebasis-da \
                       collaboraofficebasis-de \
                       collaboraofficebasis-el \
                       collaboraofficebasis-en-gb \
                       collaboraofficebasis-en-us \
                       collaboraofficebasis-eo \
                       collaboraofficebasis-es \
                       collaboraofficebasis-eu \
                       collaboraofficebasis-fi \
                       collaboraofficebasis-fr \
                       collaboraofficebasis-gl \
                       collaboraofficebasis-he \
                       collaboraofficebasis-hr \
                       collaboraofficebasis-hu \
                       collaboraofficebasis-id \
                       collaboraofficebasis-is \
                       collaboraofficebasis-it \
                       collaboraofficebasis-ja \
                       collaboraofficebasis-ko \
                       collaboraofficebasis-lo \
                       collaboraofficebasis-nb \
                       collaboraofficebasis-nl \
                       collaboraofficebasis-oc \
                       collaboraofficebasis-pl \
                       collaboraofficebasis-pt \
                       collaboraofficebasis-pt-br \
                       collaboraofficebasis-ru \
                       collaboraofficebasis-sk \
                       collaboraofficebasis-sl \
                       collaboraofficebasis-sq \
                       collaboraofficebasis-sv \
                       collaboraofficebasis-tr \
                       collaboraofficebasis-uk \
                       collaboraofficebasis-vi \
                       collaboraofficebasis-zh-cn \
                       collaboraofficebasis-zh-tw && \
# Install brand package (or not)
    if [ -z "$nobrand" ]; then \
        if [ "$type" = "cool" ] || [ "$type" = "key" ]; then \
            apt-get -y install collabora-online-brand; \
        else \
            apt-get -y install code-brand; \
        fi; \
    fi && \
# --- ADDED: Ensure template directory exists and has correct owner ---
# Create the path just in case the package install missed it, and ensure ownership
    mkdir -p /opt/collaboraoffice/share/template/common/presnt/ && \
    chown -R cool:cool /opt/collaboraoffice/share/template && \
# --- END ADDED SECTION ---
# --- PERMISSION FIX START ---
# Ensure the cool user/group exists before potentially modifying it (apt should create it, but belt-and-suspenders)
    if ! getent group cool > /dev/null; then groupadd -r -g 109 cool; fi && \
    if ! getent passwd cool > /dev/null; then useradd -r -g cool -u 109 -s /sbin/nologin -d /opt/cool cool; fi && \
# Change cool uid/gid to 1001/1001
    usermod -u 1001 cool && groupmod -g 1001 cool && \
# Recursively set ownership of /opt/cool AFTER changing the user/group ID
# This ensures the runtime user (1001) owns everything needed for jailing, cache etc.
    chown -R 1001:1001 /opt/cool && \
# Update ownership of the template dir again using NUMERIC IDs now that user cool is 1001
    chown -R 1001:1001 /opt/collaboraoffice/share/template && \
# Set ownership for config files/dirs using numeric IDs for robustness
    chown 1001:1001 /etc/coolwsd/coolwsd.xml && \
    chown 1001:1001 /etc/coolwsd && \
# --- PERMISSION FIX END ---
# Remove packages which were only needed for installation (dependecies of curl, gnupg, fc-cache etc.)
    dpkg --purge --ignore-depends=apt-transport-https,cpio,curl,dirmngr,fontconfig,fontconfig-config,fonts-dejavu-core,gnupg,gnupg-l10n,gnupg-utils,gnupg2,gpg,gpg-agent,gpg-wks-client,gpg-wks-server,gpgconf,gpgsm,krb5-locales,libassuan0,libbrotli1,libcap2-bin,libcurl4,libexpat1,libfontconfig1,libfreetype6,libgpm2,libksba8,libldap-2.5-0,libldap-common,libncursesw6,libnghttp2-14,libnpth0,libpng16-16,libpsl5,libreadline8,librtmp1,libsasl2-2,libsasl2-modules,libsasl2-modules-db,libsqlite3-0,libssh2-1,libssl3,openssl,pinentry-curses,publicsuffix,readline-common apt-transport-https cpio curl dirmngr fontconfig-config fonts-dejavu-core fontconfig gnupg gnupg-l10n gnupg-utils gnupg2 gpg gpg-agent gpg-wks-client gpg-wks-server gpgconf gpgsm krb5-locales libassuan0 libbrotli1 libcap2-bin libcurl4 libexpat1 libfontconfig1 libfreetype6 libgpm2 libksba8 libldap-2.5-0 libldap-common libncursesw6 libnghttp2-14 libnpth0 libpng16-16 libpsl5 libreadline8 librtmp1 libsasl2-2 libsasl2-modules libsasl2-modules-db libsqlite3-0 libssh2-1 pinentry-curses publicsuffix readline-common && \
# Remove perl-base, it's not needed and triggered some license scanner because of Paul Hsieh derivative license
    dpkg --purge  --force-remove-essential --ignore-depends=perl-base perl-base && \
# Cleanup
    rm -rf /var/lib/apt/* && \
# Remove WOPI Proof key generated by the package, we need unique key for each container
    rm -rf /etc/coolwsd/proof_key*

# FIXME
# Use the old starter script until we find out how to replace it.
COPY /scripts/start-collabora-online.sh /
 
RUN chown -R 1001:1001 /opt/collaboraoffice/share/Scripts/python


# coolwsd listens on port 9980
EXPOSE 9980

# Switch to cool user (use numeric user id to be compatible with Kubernetes Pod Security Policies)
USER 1001

# FIXME
# Use the old starter script until we find out how to replace it.
ENTRYPOINT ["/start-collabora-online.sh"]