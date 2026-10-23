---
description: Kör orkestern på detta repo självt, med rotens kärna som installerad version
argument-hint: "[område eller issuenummer]"
---

Begärt område: $ARGUMENTS

Läs och följ [AI.md](../../AI.md), repots värdpolicy. Läs därefter kärnan direkt från
roten: [ORCHESTRA.md](../../ORCHESTRA.md), [ISSUE-WORKFLOW.md](../../ISSUE-WORKFLOW.md)
och [COORDINATION.md](../../COORDINATION.md) för ett pass utan område eller ett
uttryckligen godkänt köpass. Utan begärt område väljer dirigenten själv valbara issues
i ett avgränsat pass utan timer och frågar inte vilket issue som ska tas.
Använd [WORKER.md](../../WORKER.md) vid delegering.

Repot konsumerar sig självt: arbetskopians rotfiler är den installerade kärnan och det
finns ingen `.orchestra/shared`. Behåll användarens avgränsning; att läsa eller underhålla
filerna startar ingen obevakad kö.
