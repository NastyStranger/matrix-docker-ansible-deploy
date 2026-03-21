# 🛡️ Managing Secrets with Ansible Vault

To keep your sensitive data (passwords, keys, tokens) secure, you should use an encrypted `vault.yml` file instead of storing secrets in plain text. This ensures that even if your configuration is shared, your credentials remain protected.

## 1. Basic Commands

Use these commands to manage your encrypted vault file:

* **Create a new Vault:**
    ```bash
    EDITOR=nano ansible-vault create inventory/host_vars/matrix.example.com/vault.yml
    ```
* **Edit an existing Vault:**
    ```bash
    EDITOR=nano ansible-vault edit inventory/host_vars/matrix.example.com/vault.yml
    ```
* **View Vault content without editing:**
    ```bash
    ansible-vault view inventory/host_vars/matrix.example.com/vault.yml
    ```

---

## 2. Implementation Workflow

### Step A: Define secrets in `vault.yml`
Store your sensitive data using a unique `vault_` prefix to distinguish them from public variables:
```yaml
vault_postgres_connection_password: "your_extremely_strong_password"
vault_matrix_domain: "example.com"
```

### Step B: Reference secrets in `vars.yml`
Link the public variables to your encrypted vault values using Jinja2 syntax:
```yaml
postgres_connection_password: "{{ vault_postgres_connection_password }}"
matrix_domain: "{{ vault_matrix_domain }}"
```

---

## 3. Pre-Deployment Checks (Pro Tips)

Before applying changes to your production server, it is highly recommended to perform a "Dry Run" and a "Syntax Check". This allows you to catch errors early and ensure that your configuration is correct without affecting the live services.

* **Step 1: Syntax Validation**
  Use this to check for typos, indentation errors, or missing quotes in your `vars.yml` and `vault.yml`:
  ```bash
  ansible-playbook -i inventory/hosts setup.yml --syntax-check --ask-vault-pass
  ```

* **Step 2: Dry Run (Simulated Deployment)**
  If the syntax is correct, use the `--check` (or `-C`) flag to simulate the actual deployment. Ansible will connect to the server and report what *would* change, without actually making any modifications:
  ```bash
  ansible-playbook -i inventory/hosts setup.yml --tags=setup-all --check --diff --ask-vault-pass
  ```
  * `--check`: Runs a simulation (Dry Run).
  * `--diff`: Shows exactly which lines in the configuration files will be modified.

---

## 4. Running the Playbook

Since your secrets are encrypted, Ansible requires a decryption key to process the configuration. You must provide the Vault password during the execution.

### Deployment Command
To apply your configuration, use the `--tags=setup-all` flag along with `--ask-vault-pass`. You will be prompted to enter your secret password before the playbook starts:

```bash
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all --ask-vault-pass
```
