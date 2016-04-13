# PokéNaXtos - https://ctf.sthack.fr

### Intro

```
Author : ShoxX - Difficulty : hard
Le professeur Chen a publié une nouvelle version du pokématos.. Que peut il bien cacher ?
```

### Checksec

```
➜ gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial

➜ cat /proc/sys/kernel/randomize_va_space 
0
```

### Binary flow analysis

```
int main() {
    esp = (esp & 0xfffffff0) - 0x20;
    do {
            puts(0x80489c8);
            puts("|  1) Consulter la map        |");
            puts(0x8048a08);
            puts("|  3) Apeller Maman           |");
            puts("_______________________________");
            stack[2039] = 0x8048a4c;
            if ((__isoc99_scanf(stack[2039], esp + 0x18, esp + 0x17) != 0x2) || ((*(int8_t *)(esp + 0x17) & 0xff) != 0xa)) {
                break;
            }
            printf(0x8048a51, stack[2039]);
            if (0x8048a51 != 0x2) {
                    if (0x8048a51 <= 0x2) {
                            if (0x8048a51 == 0x1) {
                                    map();
                            }
                    }
                    else {
                            if (0x8048a51 != 0x3) {
                                    if (0x8048a51 == 0x29) {
                                            cheat();
                                    }
                            }
                            else {
                                    call();
                            }
                    }
            }
            else {
                    consulter();
            }
    } while (true);
    return 0x0;
}

int cheat() {
    puts("____________ Cheat ___________");
    puts("|                             |");
    puts("|  Enter password :           |");
    __isoc99_scanf(0x8048978, var_1C);
    puts("_______________________________");
    if (strlen(var_1C) == 0x9) {
            if (strcmp(var_1C, "pik4_p455") == 0x0) {
                    eax = secret_place();
            }
            else {
                    printf("Bad Password !");
                    eax = *stdout@@GLIBC_2.0;
                    eax = fflush(eax);
            }
    }
    else {
            printf("Bad Password !");
            eax = *stdout@@GLIBC_2.0;
            eax = fflush(eax);
    }
    return eax;
}

int secret_place() {
    printf("Ooooh Secret Place !");
    printf("Enter cheat code :");
    eax = *stdin@@GLIBC_2.0;
    fflush(eax);
    getchar();
    gets(var_6C);
    eax = puts("Cheat Activated !");
    return eax;
}
```
