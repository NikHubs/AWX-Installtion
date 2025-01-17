
# Installationsanleitung für AWX mit K3s

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

## Installieren von AWX

### 1. Persistent Volume Claim erstellen
Erstellen Sie die Datei `public-static-pvc.yaml`:

```bash
vim public-static-pvc.yaml
```

Fügen Sie den folgenden Inhalt ein:
```yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: public-static-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 20Gi
```

Speichern Sie die Datei mit `ESC` und `:wq`.

### 2. AWX Deployment erstellen
Erstellen Sie die Datei `awx-instance-deployment.yml`:

```bash
vim awx-instance-deployment.yml
```

Fügen Sie den folgenden Inhalt ein:
```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  projects_persistence: true
  projects_storage_access_mode: ReadWriteOnce
  web_extra_volume_mounts: |
    - name: static-data
      mountPath: /var/lib/projects
  extra_volumes: |
    - name: static-data
      persistentVolumeClaim:
        claimName: public-static-data-pvc
```

Speichern Sie die Datei mit `ESC` und `:wq`.

### 3. Konfiguration anwenden
Wenden Sie die Konfigurationen an und überprüfen Sie den Status:

```bash
kubectl apply -f public-static-pvc.yaml -n awx
kubectl apply -f awx-instance-deployment.yml -n awx
kubectl get pods -n awx
```

### 4. Status live verfolgen
Verfolgen Sie den Fortschritt mit folgendem Befehl (mit `STRG+C` verlassen):

```bash
kubectl get pods -l "app.kubernetes.io/managed-by=awx-operator" -n awx -w
```

---

## Zugriff auf AWX

### 1. Service überprüfen
Finden Sie die IPs und Ports mit folgendem Befehl:

```bash
kubectl get svc -n awx
```

Beispielausgabe:
```plaintext
[root@awx-server]# kubectl get svc -n awx
NAME                                              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
awx-operator-controller-manager-metrics-service   ClusterIP   10.43.219.250   <none>        8443/TCP                195d
awx-postgres-13                                   ClusterIP   None            <none>        5432/TCP                195d
awx-service                                       NodePort    10.43.154.175   <none>        80:30080/TCP            195d
```

- **Port 30080** ist für die Web-Oberfläche vorgesehen (Port 80 ist für K3s reserviert).

### 2. Port ändern (optional)
Falls Sie den Port ändern möchten, verwenden Sie folgenden Befehl:

```bash
kubectl patch svc awx-service -n awx --type='json' -p='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value": 30080}]'
```

### 3. Passwort für Anmeldung finden
Finden Sie das Standard-Passwort mit folgendem Befehl:

```bash
kubectl -n awx get secret awx-admin-password -o go-template='{{range $k,$v := .data}}{{printf "%s: " $k}}{{if not $v}}{{$v}}{{else}}{{$v | base64decode}}{{end}}{{"\n"}}{{end}}'
```

### 4. Anmeldung
- Öffnen Sie die AWX-Web-Oberfläche unter: `http://<SERVER-IP>:<PORT>`.
- Der Standard-Benutzername ist `admin`.
- Das Passwort finden Sie im letzten Schritt.

---

## Zusammenfassung
Mit dieser Anleitung haben Sie:
- **K3s installiert und konfiguriert.**
- **Den AWX Operator bereitgestellt.**
- **AWX erfolgreich installiert und Zugriff erhalten.**

Falls Sie weitere Unterstützung benötigen, werfen Sie einen Blick in die [K3s-Dokumentation](https://k3s.io) oder die [AWX Operator GitHub-Seite](https://github.com/ansible/awx-operator).
