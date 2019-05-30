FROM protecto/bioweb

RUN sudo apt-get update && sudo apt-get install -y \
        vim \
        nano \
        emacs \
    && sudo rm -rf /var/lib/apt/lists/*

RUN printf '#!/bin/bash\n\nset -ex\nsudo service postgresql start\n/bin/bash $*' > /home/developer/entrypoint \
    && chmod +x /home/developer/entrypoint

WORKDIR /home/developer/bioweb

EXPOSE 9000

ENTRYPOINT ["/home/developer/entrypoint"]
