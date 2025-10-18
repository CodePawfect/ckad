## Aufgabe

Erstelle einen CronJob der:

- Jede Minute läuft (`*/1 * * * *`)
- Command `date` ausführt
- Nach **22 Sekunden terminiert** wird
- Name: `hello` (CronJob und Container)
- Image: `busybox`

### Methode 1: YAML generieren und editieren (empfohlen)

```bash
# 1. Generiere Basis-YAML
kubectl create cronjob hello --image=busybox \
  --schedule="*/1 * * * *" \
  --dry-run=client -o yaml -- date > cron.yaml

# 2. Editiere die Datei
vim cron.yaml

# 3. Füge activeDeadlineSeconds hinzu (siehe unten)

# 4. Erstelle CronJob
kubectl apply -f cron.yaml
```

### Das vollständige YAML

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      activeDeadlineSeconds: 22  # ← DIESE ZEILE MANUELL HINZUFÜGEN!
      template:
        spec:
          containers:
          - command:
            - date
            image: busybox
            name: hello
          restartPolicy: OnFailure
```

**WICHTIG:** Die Zeile `activeDeadlineSeconds: 22` muss auf der **gleichen Einrückungsebene** wie `template:` stehen (unter `spec.jobTemplate.spec`)!

|Feld|Ebene|Bedeutung|
|---|---|---|
|`activeDeadlineSeconds`|`jobTemplate.spec`|Job wird nach X Sekunden **terminiert** ✅|
|`startingDeadlineSeconds`|`spec`|Job gilt als "verpasst" wenn er nicht innerhalb X Sekunden **startet** ❌|

#flashcards/ckad 
Welches Feld um im cronjob die akive Zeit bis zu einem Timeout einzustellen?
?
spec:
  jobTemplate:
    spec:
       activeDeadlineSeconds: 22