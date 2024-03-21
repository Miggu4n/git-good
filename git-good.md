# Git good

Git è un sofwate necessario per il **versionamento del codice**. Saper utilizzare git al meglio è una skill fondamentale per mantenere un flusso di lavoro pulito, prevedibile e che permette di ottenere il massimo dal lavoro di squadra.

Questa guida vuole essere il punto d'inizio di una documentazione su un nostro flusso di sviluppo.

**Requisiti:**

- bisogna avere già delle minime conoscenze di git e un servizio tipo github.

## Ticket

Un ticket è un'obiettivo che lo sviluppatore deve raggiungere. Questo obiettivo viene creato dal PM su proposta del committente. Nel caso del portale [deeealer.com](https://deeealer.com), il committente è [Federica Sartorel](mailto:federicasartorel@tomasiauto.com).

Ogni ticket è composto da un codice identificativo univoco, un titolo e una descrizione. Il PM crea un ticket sulla board jira del progetto e lo assegna ad uno sviluppatore.

Lo sviluppatore prende in carico l'obiettivo e apre il repository del progetto sul suo editor preferito.

## Allineamento

Ora bisogna capire da che versione del codice arriva la segnalazione. Se Federica riscontra il problema in fase di testing, bisognerà modificare la versione del codice pubblicata in ambiente di testing.
Viceversa, se la segnalazione arriva dall'ambiende di produzione (es: un cliente deeealer non riesce ad aggiungere un veicolo al carrello), la versione del codice da testare e fixare è quella di produzione.

Una volta capito da che versione del codice bisogna partire, si effettua un checkout sul branch dedicato. Prendiamo in considerazione la seconda casistica descritta prima. L'errore proviene da una segnalazione in produzione, quindi ci si allinea al branch main

    git checkout main

> Prima di fare un checkout sarebbe buona prassi [controllare se ci sono versioni più recenti di quella presente in locale](https://git-scm.com/docs/git-fetch).

## Sviluppo

Siccome dobbiamo tenere traccia delle modifiche, non andremo a toccare il codice direttamente in main ma faremo un checkout su un nuovo branch che chiameremo _feature branch_. Sarebbe buona prassi utilizzare il codice identificativo del ticket come nome del feature branch.

    git checkout -b [ticket_id]

Nel nostro caso, il ticket ha un codice univoco uguale a B2B-123. D'ora in poi ricordiamoci questo codice come _ticket_id_
La struttura dovrebbe simile essere questa:

```
    * staging
    | * B2B-123
    |/
    * main
```

> Nel caso in cui la segnalazione arrivasse da staging, la struttura sarebbe la seguente:
>
> ```
>   * B2B-123
>   |
>   * staging
>   |
>   * main
> ```

Da qui si prova a riprodurre l'errore segnalato. Una volta riprodotto, si fanno modifiche al codice per trovare le soluzioni. In questa fase è importante fare commit piccoli e concisi. Spesso si crede che tanti commit sporchino il grafico. In realtà quello che sporca il grafico non sono i troppi commit ma un utilizzo di git da principianti.

Un'altra cosa da evitare è l'utilizzo ossessivo-compulsivo degli stash. Negli stash dovrebbero finire modifiche che non vogliamo nel nostro codice ma che ci fa quasi pensa scartare. Esempio: un piccolo script o del codice che si potrebbe tranquillamente commentare.

## Testing

Una volta sistemato il problema e testato il tutto in locale, le nostre modifiche si devono unire con la versione più recente del codice in staging.

    $ git checkout staging
    $ git fetch
    $ git pull
    $ git merge ticket_id --no-ff

il flag _--no-ff_ ci permette di fornire **sempre** un messaggio di merge.

Siccome più avanti utilizzeremo la libreria [generate-changelog](https://www.npmjs.com/package/generate-changelog), questo messaggio dovrebbe essere formattato con una sintassi specifica. Entreremo nel dettaglio più avanti, per ora è sufficiente ricordarsi che quando si uniscono le modifiche di un feature branch in staging sarebbe buona prassi fornire un messaggio di merge.

Il PM ora prova il fix su staging. Se tutto è ok si può passare al prossimo obiettivo assegnato, partendo dal branch di segnalazione, sviluppando le modifiche e unendo le modifiche nel branch di testing.

## Pre rilascio

Dopo un determinato numero di obiettivi raggiunti si effettua un rilascio. Il rilascio consiste nell'aggiornare il codice nel branch di produzione (in questo caso, main).

Riallacciandoci alla sintassi del commit di merge da feature-branch a staging, il messaggio deve essere formattato in questo modo:

```
type(category): description [flags]
```

> Il motivo di questa sintassi è ben documentato: [generate-changelog docs](https://github.com/lob/generate-changelog#readme)

Ci spostiamo sul branch di produzione e ci prepariamo ad importare le modifiche in staging.
In questa fase è preferibile effettuare un merge fast-forward. Il motivo di questa differenza è chiaro in questa immagine:

![alt txt](https://i.stack.imgur.com/FMD5h.png)

_--no-ff_ crea un commit di merge mentre _--ff_ prosegue sulla stessa linea

## Rilascio

Deeealer è un progetto scritto in next.js, quindi stack javascript. Per questo motivo installeremo [generate-changelog](https://github.com/lob/generate-changelog) per generare i changelog e il versionamento.
Questa libreria ci permette di:

- aggiornare in automatico il CHANGELOG.md
- creare committare le modifiche al changelog
- aggiornare la versione nel package.json in modo consistente (gestione major, minor e patches)
- creare un commit con messaggio uguale alla versione del package.json appena aggiornato (es: v1.2.7)
- create un tag con valore uguale alla versione pubblicata. I tag sono molto utili nel caso in cui si rende necessario un rollback

Una volta installata la libreria, possiamo rilasciare utilizzando uno dei seguenti comandi:

- `yarn release:patch` dove aggiorneremo il numero dedicato alle patch (es: 1.0.x)
- `yarn release:minor` dove aggiorneremo il numero dedicato alle minor releases (es: 1.x.0)
- `yarn release:patch` dove aggiorneremo il numero dedicato alle major releases (es: x.0.0)

La libreria farà tutto il necessario e effettuerà in autonomia il push in remoto.
