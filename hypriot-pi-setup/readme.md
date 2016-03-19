This playbook assumes that you already have an SSH key at `~/.ssh/id_rsa`. All steps are described in `setup.yml`.

    # Add the Raspberry Pi IP to the file `hosts`
    echo "[pi]
    192.168.0.123" > hosts

    # Run the playbook
    ansible-playbook setup.yml --user root --ask-pass
