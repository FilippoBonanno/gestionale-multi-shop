# Gestionale multi shop

Back office che governa le vendite di un'azienda di ricambi su **più canali contemporaneamente**: un negozio eBay e quattro negozi PrestaShop (B2B, B2C, Francia, Europa). Ordini, magazzino, acquisti dai fornitori e spedizioni passano tutti da qui, usato ogni giorno da circa 11 operatori.

Ho preso in carico un gestionale già esistente (PHP generato con PHPRunner, diversi anni di storia) e ne curo **manutenzione ed evoluzione**: nuovi moduli, integrazioni, analisi dei flussi, rilasci controllati.

> Il codice appartiene al cliente e non è pubblicato: posso descrivere architettura e scelte nel dettaglio durante un colloquio.

**Stack:** PHP 7.4 · MySQL · PHPRunner 9.8 (fino a settembre 2026, poi codice gestito a mano) · JavaScript · ECharts · PHPExcel / TCPDF · cron · API eBay (Trading SOAP/XML + REST, OAuth) · PrestaShop Webservice · API corrieri

## In numeri
- **5 canali** di vendita sincronizzati (eBay + 4 PrestaShop, migrati nel frattempo a PrestaShop 9)
- circa **170 tabelle** gestite, **12 processi automatici** (cron da ogni 9 minuti a giornalieri)
- **50 versioni** rilasciate in circa 7 mesi (aprile – novembre 2026), ognuna con changelog e ambiente di staging separato dalla produzione

## Cosa fa
**Ordini.** Importa gli ordini da eBay ogni 10 minuti e dai negozi PrestaShop ogni 15, in tabelle unificate; a ogni ordine genera lo scarico di magazzino. Gestisce anche ordini manuali e vendita al banco.

**Magazzino e catalogo.** Articoli padre con alias SKU e varianti, giacenza come unica fonte di verità; sincronizzazione delle quantità verso tutti i canali, con un passaggio "tempo reale" sugli ultimi movimenti. Pubblicazione degli articoli su eBay e PrestaShop con immagini, categorie e controllo anti-duplicato.

**Sottoscorta e ordini ai fornitori** *(modulo che ho progettato e sviluppato, versioni 1.25 – 1.50)*. Dal sottoscorta nasce la bozza d'ordine per fornitore; export Excel con le foto incorporate; reimport delle quantità e dei prezzi corretti dal fornitore; controllo righe al carico con gestione delle mancanze (rispedizione, ordine successivo, backorder) ed export dei reclami; dichiarazione doganale per fornitore in EUR/USD; tracking dei colli; registro delle azioni; stampa etichette SKU su stampante termica.

**Spedizioni.** Tracciati e invio spedizioni per GLS (anche via API), SDA/Poste, BRT e ARCO; inserimento tracking e chiusura ordini sui canali. **Dashboard spedizioni** interna *(sviluppata da me)* con KPI e grafici, sincronizzazione incrementale ogni 10 minuti: ha sostituito un servizio esterno a pagamento.

**Report.** Fatturato per articolo e per cliente, venduto per periodo, invenduti, articoli tornati sotto scorta, export Excel.

## Lavoro ingegneristico
- **Analisi dei flussi critici** prima di toccare il codice: import ordini, comportamento dei cron senza rete (finestre di recupero e procedura), cause delle immagini duplicate sui negozi con script di riallineamento.
- **Rilasci sicuri:** staging con database separato, checklist dei file che non vanno mai caricati, controllo delle terminazioni di riga, rilasci in blocco documentati.
- **Performance:** sincronizzazione della dashboard incrementale con lock, limiti di paginazione sulle liste pesanti, un solo processo autorizzato a ricalcolare i flag di giacenza.
- **Compatibilità PHP 8:** censimento di circa 900 chiamate alla vecchia estensione mysql e correzioni preparatorie.
- **Riscrittura in corso** su architettura moderna (NestJS + PostgreSQL + Redis/BullMQ, React + Vite, adapter per ogni canale, Docker): motivata da un'analisi del codice esistente (funzioni duplicate per negozio, rami morti di vecchi clienti, file generati non versionabili). Connessioni PrestaShop ed eBay già provate in sandbox.
- **Guide per gli operatori** scritte da me (sottoscorta, stampante etichette).

---
**Filippo Bonanno** · sviluppatore web (PHP, PrestaShop, Laravel, JavaScript) · altri progetti: [moduli PrestaShop](https://github.com/FilippoBonanno/prestashop-modules-showcase), [caso di studio e-commerce](https://github.com/FilippoBonanno/abtecnology-prestashop)
