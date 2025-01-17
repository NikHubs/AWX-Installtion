
# Installationsanleitung für K3s und AWX Operator

## Vorbereitungen

### 1. Linux-System aktualisieren
Um sicherzustellen, dass Ihr System bereit ist, bringen Sie es auf den neuesten Stand. Führen Sie dazu folgenden Befehl aus:

```bash
apt-get update && apt-get upgrade -y
```

---

## Installieren von K3s

### Voraussetzungen
- **Internetverbindung:** Für die Installation wird eine aktive Internetverbindung benötigt.
- **Root-Benutzer:** Die Installation sollte idealerweise als Root-Benutzer durchgeführt werden.

### Installation von K3s
Laden Sie K3s herunter und installieren Sie es mit folgendem Befehl:

```bash
curl -sfL https://get.k3s.io | sh -
```

### Status überprüfen
Nach der Installation können Sie den Status von K3s überprüfen:

```bash
systemctl status k3s
```

Überprüfen Sie, ob die Nodes korrekt eingerichtet wurden:

```bash
kubectl get nodes
```

### Erfolgsmeldung
Mit diesen Schritten haben Sie K3s erfolgreich installiert, und es sollte nun laufen.

---

## Installieren des AWX Operators

### Voraussetzungen
Der **AWX Operator** wird ab AWX-Version **<18.0.0** für die Installation benötigt.

### Schritte zur Installation
Führen Sie die folgenden Befehle aus, um den AWX Operator zu installieren:

1. Klonen Sie das AWX Operator Repository:
   ```bash
   git clone https://github.com/ansible/awx-operator.git
   ```

2. Erstellen Sie einen Namespace für AWX:
   ```bash
   export NAMESPACE=awx
   kubectl create ns ${NAMESPACE}
   ```

3. Wechseln Sie in das AWX Operator Verzeichnis:
   ```bash
   cd awx-operator/
   ```

4. Finden Sie den neuesten Release-Tag und checken Sie diesen aus:
   ```bash
   RELEASE_TAG=`curl -s https://api.github.com/repos/ansible/awx-operator/releases/latest | grep tag_name | cut -d '"' -f 4`
   echo $RELEASE_TAG
   git checkout $RELEASE_TAG
   ```

5. Deployen Sie den AWX Operator:
   ```bash
   make deploy
   ```

### Status überprüfen
Die Installation kann einige Minuten dauern. Überprüfen Sie den Status der Pods mit:

```bash
kubectl get pods -n awx
```

---

## Zusammenfassung
- **K3s** wurde erfolgreich installiert und läuft.
- Der **AWX Operator** wurde bereitgestellt und kann verwendet werden.

Falls Sie weitere Unterstützung benötigen, werfen Sie einen Blick in die [K3s-Dokumentation](https://k3s.io) oder die [AWX Operator GitHub-Seite](https://github.com/ansible/awx-operator).
