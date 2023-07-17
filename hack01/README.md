# Syssoft - Projeto Final

## Exercício 1 - Resolução

O primeiro passo tomado pela equipe foi descobrir como o executável *docrypt* funcionava. Para isso, foram utilizados algumas ferramentas, como por exemplo o *readelf*:

```
readelf -d docrypt
```

O retorno obtido nos deu uma pista importante: *docrypt* precisa de duas *shared library*, em especial a *libauth*.

```
  Tag        Type                         Name/Value
 0x00000001 (NEEDED)                     Shared library: [libauth.so]
 0x00000001 (NEEDED)                     Shared library: [libc.so.6]
```

Com isso, passamos a investigar mais a fundo o programa *docrypt*, através da ferramenta *objdump*.

```
objdump -D docrypt | less
```

O que descobrimos foi que, na função *main* desse executável, havia uma chamada para uma função com um nome suspeito: **_authorize_**.

```
 138f:       e8 ac fd ff ff          call   1140 <authorize@plt>
```

Logo percebemos que essa função estava presente na *library* *libauth*, e que poderiamos evitar a chamada da função original carregando uma *shared library* antes da *libauth* e que possuisse uma função com o mesmo nome. Se a função fizesse o que o seu nome sugeria, conseguiríamos passar pelo sistema de autenticação.

Foi criada então uma *shared library* com o nome de *libauthpass*, com o seguinte código dentro:

```C
int authorize() {
	return 1;
}
```

Para gerar a *shared library*:

```
gcc --shared -m32 authorize.c -o libauthpass.so
```

Por fim, foi realizado um teste para verificar se tinhamos conseguido burlar o sistema de autenticação. O resultado foi um sucesso.

```
LD_PRELOAD=libauthpass.so ./docrypt test.cry
User name: 1
Authorization key: 1
Decryption key: test
Hello World
```

Logo, bastava descobrir a chave de encriptação. Para isso, investigamos o executável *encrypt* com a ferramenta *objdump*:

```
objdump -D encrypt | less
```

Verificamos na seção *.rodata*, uma variável tinha um nome peculiar: **_encrypt_key_**:

```
00002024 <encrypt_key>:
    2024:       65 61                   gs popa 
    2026:       73 79                   jae    20a1 <__GNU_EH_FRAME_HDR+0x75>
```

Convertendo o código hexadecimal para ASCII, o valor para *encrypt_key* encontrado foi: **_easy_**.

Assim, com o chave em mãos e o sistema de autenticação burlado, conseguimos descobrir o conteudo do arquivo *sample.cry*:

```
LD_PRELOAD=libauthpass.so ./docrypt sample.cry 
User name: 1
Authorization key: 1
Decryption key: easy

   Excerpt of RFC-1392 (Request for Comments)
   -------------------------------------------------------------

   Network Working Group                           G. Malkin
   Request for Comments: 1392                      Xylogics, Inc.
   FYI: 18                                         T. LaQuey Parker
                                                   UTexas
                                                   Editors
                                                   January 1993


      			        (...)

                        Internet Users' Glossary


   hacker
      A person who delights in having an intimate understanding of the
      internal workings of a system, computers and computer networks in
      particular.  The term is often misused in a pejorative context,
      where "cracker" would be the correct term.  See also: cracker.

```

## Exercício 1 - Como executar

Para executar a descriptografia de um arquivo, execute a sequencia de passos a seguir:

1. Crie a *shared library* (é necessário realizar esta etapa uma única vez)
   
   ```
   make
   ```

2. Execute o comando *make run* a seguir:
   
   ```
   make run FILE=<input> KEY=<key>
   ```
   
   - input: arquivo a ser descriptografado;
   - key: chave de encriptação - a chave encontrada foi **easy**

4. Escolha qualquer usuário e senha para poder acessar o arquivo.