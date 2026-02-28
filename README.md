# Archipel

Prototype P2P local, chiffre, sans serveur central.

## Etat du depot (constate localement)

Ce repository contient:

- un transport TCP chiffre avec handshake (`HELLO -> HELLO_REPLY -> AUTH -> AUTH_OK`)
- une decouverte de pairs en UDP multicast (`239.255.42.99:6000`)
- un transfert de fichiers en chunks (`512 KB`) avec manifest + ACK
- un stockage local des chunks dans `.archipel/`
- une CLI de demo dans `src/cli/main.py`
- un utilitaire Sprint 0 dans `sprintO.py`

Fonctionnalites encore en placeholder:

- `download` dans la CLI principale
- `status` dans la CLI principale
- `src/messaging/service.py`
- `demo/run-demo.ps1` (TODO)

## Arborescence utile

```text
src/
  cli/main.py
  crypto/{keys.py,session.py,trust_store.py}
  network/{constants.py,packet.py,discovery.py,tcp_server.py,tcp_client.py,wifi_direct.py,peer_table_sprint1.py}
  transfer/{chunking.py,manifest.py,chunk_store.py}
docs/
  architecture.md
  protocol-spec.md
tests/
  test_peer_table.py
  test_placeholder.py
  test_e2e_sprint2.py
sprintO.py
pyproject.toml
```

## Prerequis

- Python `>= 3.10`
- Windows (le module `wifi_direct.py` est oriente PowerShell/Windows)

Dependances Python utilisees par le code:

- `pynacl`
- `pycryptodome` (module `Crypto.Cipher.AES`)
- `cryptography` (utilise par `sprintO.py`)
- `pytest` (pour les tests)

`pyproject.toml` ne declare actuellement que `pynacl`, donc il faut installer le reste manuellement.

## Installation rapide

```bash
python -m venv venv
venv\Scripts\activate
pip install -e .
pip install pycryptodome cryptography pytest
```

## Commandes principales

Generer les cles Ed25519 pour la CLI principale:

```bash
python -m src.cli.main keygen
```

Demarrer un noeud TCP:

```bash
python -m src.cli.main start --port 7777
```

Lister les pairs connus (`.archipel/peers.json`):

```bash
python -m src.cli.main peers
```

Envoyer un message chiffre:

```bash
python -m src.cli.main msg <node_id_hex> "Bonjour" --ip <ip_peer> --port 7777
```

Envoyer un fichier (manifest + chunks):

```bash
python -m src.cli.main send <chemin_fichier> --ip <ip_peer> --port 7777
```

Lister les fichiers recus (chunk store):

```bash
python -m src.cli.main receive
```

Marquer un pair comme trusted (TOFU):

```bash
python -m src.cli.main trust <node_id_hex>
```

Gestion reseau "ile" locale (Wi-Fi Direct / hotspot):

```bash
python -m src.cli.main network create-island
python -m src.cli.main network status
```

## Service de decouverte UDP (separe)

La commande `start` n'active pas automatiquement la decouverte multicast.
Pour lancer la discovery:

```bash
python -m src.network.discovery --node-id-file keys/ed25519_public.key --tcp-port 7777
```

## Utilitaire Sprint 0 (`sprintO.py`)

```bash
python sprintO.py --help
python sprintO.py keygen --node node-1 --out keys
python sprintO.py packet-demo --node-id-file keys/node-1_node_id.bin
python sprintO.py report --node node-1 --keys-dir keys
```

## Tests

Tests unitaires:

```bash
pytest -q tests/test_peer_table.py tests/test_placeholder.py
```

Script e2e local (non pytest):

```bash
python tests/test_e2e_sprint2.py
```

## Fichiers de donnees et runtime

- `.archipel/peers.json`: table des pairs
- `.archipel/trust_store.json`: statut trust TOFU
- `.archipel/chunks/`: chunks recus + manifests
- `downloads/`: fichiers reassembles
- `keys/`, `keys_a/`, `keys_b/`: cles de test

## Notes

- `combinee.pdf` et `test_50mb.bin` sont presents a la racine pour les essais de transfert.
- Date de mise a jour du README: 2026-02-28.
