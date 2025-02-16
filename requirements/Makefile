all: install_ansible install_collections
.PHONY: all

install_ansible:
	pip install -U -r requirements.txt

install_collections:
	ansible-galaxy install -r galaxy-requirements.yaml --force-with-deps
