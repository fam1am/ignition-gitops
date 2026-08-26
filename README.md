# Fares Metwally | 8/26/2026

Note: This guide assumes you are on a Linux machine and have Docker + Git installed

1) Clone into your home dir

    cd ~
    git clone https://github.com/<org>/<project-name>.git
    cd ~/<project-name>
    git checkout main

2) Match Ignition to this host's user

    export IGNITION_UID=$(id -u)
    export IGNITION_GID=$(id -g)

    // persist for later shells (optional)
    echo "export IGNITION_UID=$(id -u)" >> ~/.bashrc
    echo "export IGNITION_GID=$(id -g)" >> ~/.bashrc

3) First boot: volume only (init.yaml)

    docker compose -f development/init.yaml up -d
    docker compose -f development/init.yaml ps

    Open http://<host>:8088 and finish commissioning if this is a fresh volume.

4) Switch to live binds (compose.yaml)

    docker compose -f development/init.yaml down     # keeps the volume; do not add -v
    docker compose -f development/compose.yaml up -d
    docker compose -f development/compose.yaml ps

    Same project name (ignition-dev), so this replaces gateway-init.
    Binds: ../config/... → git-controlled tags and UDTs.

* Day-to-day (from ~/<project-name>, with IGNITION_UID/GID still set):

    docker compose -f development/compose.yaml up -d
    docker compose -f development/compose.yaml logs -f gateway
    docker compose -f development/compose.yaml stop
    docker compose -f development/compose.yaml down      # keeps ignition-dev-data

* After Designer edits (they land in ./config, which is the git tree):

    cd ~/<project-name>
    git status
    git diff
    git add config
    git commit -m "Update gateway tags from Designer"
    git push origin main

* To pick up someone else's changes:

    git pull origin main
    // no compose recreate needed; binds are live
