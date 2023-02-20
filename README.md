# Anteckningar 2022-02-20
Implementering av PCI-avbrott för egenskapad processor, vilket möjliggör avbrott vid logisk förändring av insignaler. För detta ändamål implementeras monitorering av insignaler på samtliga I/O-portar (via innehållet i pin-register PINB - PIND). Monitorering sker under varje klockpuls, medan kontroll efter IRQ (interrupt requests) enbart genomförs i slutet av varje EXECUTE-fas.

Även instruktioner SEI, CLI samt RETI implementeras för att kunna aktivera respektive inaktivera avbrott samt genomföra återhopp efter slutfört avbrott.

För en given insignal gäller att ifall motsvarande maskbit har ettställs i tillhörande maskregister PCMSK0 - PCMSK2 jämförs nuvarande insignal mot föregående insignal. Om dessa är olika ettställs motsvarande interrupt-flagga PCIF0 - PCIF2 i flaggregistret PCIFR för att generera en interrupt request (IRQ), vilket innebär att aktuell avbrottskälla väntar på
att få generera ett avbrott.

Vid kontroll efter eventuella IRQ kontrolleras först och främst så att I-flaggan är ettställd (den globala interrupt-flaggan i statusregistret). Om detta är fallet kontrolleras interrupt-flaggorna PCIF0 - PCIF2 i flaggregistret PCIFR. Om en given interrupt-flagga PCIF0 - PCIF2 är ettställd samtidigt som motsvarande enable-bit PCIE0 - PCIE2 i kontrollregistret PCICR är ettställd genereras ett avbrott. Enbart ett avbrott genereras åt gången, så är fler interrupt-flaggor samt motsvarande enable-bitar ettställda kommer dessa att behandlas först när aktuellt avbrott är slutfört.

Innan ett avbrott genereras nollställs interrupt-flaggan PCIF0 - PCIF2 i fråga samt I-flaggan, den förra för att terminera
aktuell IRQ (annars kommer avbrottet genereras igen), den senare för att eventuellt ytterligare avbrott inte ska kunna
äga rum förrän aktuellt avbrott har terminerats. Returadressen efter avbrottet (programräknarens innehåll) sparas på stacken. Programhopp sker sedan till motsvarande avbrottsvektor. När avbrottet avslutas hämtas returadressen från stacken.

Avbrottsimplementeringen testas via ett program, där en lysdiod togglas vid nedtryckning av en tryckknapp. PCI-avbrott har aktiverats på tryckknappen i fråga så att programhopp sker till motsvarande avbrottsrutin PCINT0_vect vid nedtryckning.

Samtliga .c- och .h-filer utgörs av processorn med färdig interrupt-implementering.