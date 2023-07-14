# Misc notes

## Pack and ship Python virtual environment

- Problem: We are not allowed to install anything on the remote server where data are generated. 
- Solution: Build a `ds_virtualenv` locally with most common Python packages. This enviorment can be copied to the remote machine.
- Tool: [venv-pack](https://jcristharif.com/venv-pack/)