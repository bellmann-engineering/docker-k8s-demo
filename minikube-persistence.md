### 1. Minikube starten:

Stelle sicher, dass Minikube gestartet ist. Wenn nicht, führe den folgenden Befehl aus:

```bash
minikube start --driver=docker --mount=true
```
Sollte es zu einem Fehler beim Start kommen, so löscht man am besten das minikube Profil einmal vollständig:

```bash
minikube stop
minikube delete
```
danach
```bash
minikube start --driver=docker --mount=true
```

### 2. Persistent Volume (PV) und Persistent Volume Claim (PVC) erstellen:

Erstelle die YAML-Dateien für das Persistent Volume (`pv.yaml`) und den Persistent Volume Claim (`pvc.yaml`):

**pv.yaml:**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Die Konfiguration path: "/mnt/data" im Persistent Volume (PV) bezieht sich auf den Dateipfad auf der Host-Maschine, auf der der Minikube-Cluster läuft. 

**pvc.yaml:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      pv-name: my-pv
```

Führe die Konfigurationen mit den folgenden Befehlen aus:

```bash
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

### 3. Minikube Mount ausführen:

Starte den Minikube-Mount, um ein Verzeichnis vom Host in die Minikube-VM zu mounten:

```bash
minikube mount C:\data:/mnt/data
```

### 4. Deployment aktualisieren:

Ändere deine vorhandene Deployment YAML-Datei (`tinyweb-deployment.yaml`), um einen Volume und VolumeMounts für den Persistent Volume Claim einzuschließen:

**tinyweb-deployment.yaml:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tinyweb-deployment
  labels:
    app: tinyweb
spec:
  replicas: 3
  selector:
    matchLabels:
      app: tinyweb
  template:
    metadata:
      labels:
        app: tinyweb
    spec:
      containers:
      - name: tinyweb
        image: kbellmann/tinyweb:0.9
        imagePullPolicy: Always
        ports:
        - containerPort: 80
        volumeMounts:
        - name: tinyweb-storage
          mountPath: "/tmp"
      volumes:
      - name: tinyweb-storage
        persistentVolumeClaim:
          claimName: my-pvc
```

Führe die aktualisierten Konfigurationen aus:

```bash
kubectl apply -f tinyweb-deployment.yaml
```

### 5. Persistenz überprüfen:

Überprüfe, ob das Deployment ausgeführt wird, und inspiziere das Persistent Volume, den Persistent Volume Claim und die Pods:

```bash
kubectl get deployments
kubectl get pods
kubectl get pv
kubectl get pvc
```

Mit dieser Konfiguration wird der Pfad `/tmp` in jedem Pod des Deployments mit dem Host-Pfad `C:\data` auf Windows verbunden. Jeder Pod hat seinen eigenen Speicherplatz, der persistent ist, selbst wenn der Pod beendet und neu erstellt wird.
