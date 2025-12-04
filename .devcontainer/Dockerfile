ARG DEBIAN_RELEASE=trixie-slim
FROM debian:${DEBIAN_RELEASE}

RUN apt-get update && apt-get install -y locales && rm -rf /var/lib/apt/lists/* \
	&& localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8
ENV LANG=en_US.utf8

ENV TZ=Europe/Zurich
#Install the tzdata package to the configure timezone
RUN apt update && \
	apt install -y tzdata && \
	ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && \
	echo $TZ > /etc/timezone && \
	apt clean && \
	rm -rf /var/lib/apt/lists/*

RUN apt update && apt install -y coreutils zsh-common
RUN apt update && apt install -y vim tealdeer
RUN apt install -y zsh git curl wget wget2 openssh-client gpg fzf jq
RUN apt update && apt install -y man man-db && \
    apt clean && \ 
    rm -rf /var/lib/apt/lists/*

# Create non-root user "appuser" and home directory
ARG USERNAME=appuser
ARG USER_UID=1000
ARG USER_GID=1000

RUN groupadd --gid ${USER_GID} ${USERNAME} \
    && useradd --uid ${USER_UID} --gid ${USER_GID} -m --shell=/bin/zsh ${USERNAME} \
    && mkdir -p /home/${USERNAME}/workspace \
    && chown -R ${USERNAME}:${USER_GID} /home/${USERNAME}
# Creating a runtime user
RUN apt update && apt install -y sudo && rm -rf /var/lib/apt/lists/* 

#Adding user to the sudo group
RUN usermod -aG sudo appuser
# Disable sudo password for the user.
RUN echo '%sudo ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

RUN chsh -s $(which zsh)

#Switching to the user

USER ${USERNAME}
ENV PATH="/home/${USERNAME}/.local/bin:${PATH}"

WORKDIR /home/appuser

RUN mkdir -p /home/${USERNAME}/.config/vim
ENV XDG_CONFIG_DIR=/home/${USERNAME}/.config
ENV MYVIMRC=/home/${USERNAME}/.config/vim/vimrc
ENV VIMINIT='source ${MYVIMRC}'
#Install the ZSH shell for the user
# Uses "Spaceship" theme with some customization. Uses some bundled plugins and installs some more from github
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.1/zsh-in-docker.sh)" -- \
    -x \
    -t https://github.com/denysdovhan/spaceship-prompt \
    -a 'SPACESHIP_PROMPT_ADD_NEWLINE="false"' \
    -a 'SPACESHIP_PROMPT_SEPARATE_LINE="false"' \
    -p eza \
    -p fzf \
    -p git \
    -p ssh-agent \
    -p https://github.com/zsh-users/zsh-autosuggestions \
    -p https://github.com/zsh-users/zsh-completions

RUN sudo mkdir -p /etc/apt/keyrings
RUN wget -qO- https://raw.githubusercontent.com/eza-community/eza/main/deb.asc | sudo gpg --dearmor -o /etc/apt/keyrings/gierens.gpg
RUN echo "deb [signed-by=/etc/apt/keyrings/gierens.gpg] http://deb.gierens.de stable main" | sudo tee /etc/apt/sources.list.d/gierens.list
RUN sudo chmod 644 /etc/apt/keyrings/gierens.gpg /etc/apt/sources.list.d/gierens.list
RUN sudo apt update
RUN sudo apt install -y eza && sudo rm -rf /var/lib/apt/lists/* 

RUN sudo chsh -s $(which zsh)

RUN git clone https://github.com/junegunn/fzf-git.sh.git
COPY --chown=${USERNAME}:${USER_GID} .devcontainer/zsh_config.zsh /home/${USERNAME}/.oh-my-zsh/custom/
RUN sudo chmod +x /home/${USERNAME}/.oh-my-zsh/custom/zsh_config.zsh

WORKDIR /home/${USERNAME}/workspace

CMD [ "/bin/zsh" ]
