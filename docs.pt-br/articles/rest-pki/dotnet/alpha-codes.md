﻿# Gerando códigos alfanuméricos em .NET

Ao gerar uma versão para impressão de um arquivo assinado, é necessário gerar um *código de verificação* a ser
incluido no documento para que um terceiro que porventura receba o documento impresso possa visitar o seu site e fornecer o
código, obtendo assim a versão assinada digitalmente (que é a que efetivamente tem validade jurídica).

No passado, nós províamos o código-fonte para geração desse código de verificação como parte dos exemplos, a ser copiado
para o código da sua aplicação. Por exemplo:

```cs
public static class Util {
   
   // ...

   public static string GenerateVerificationCode() {
      // String with exactly 32 letters and numbers to be used on the codes.
      const string Alphabet = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789";
      // Allocate a byte array large enough to receive the necessary entropy
      var bytes = new byte[(int)Math.Ceiling(VerificationCodeSize * 5 / 8.0)];
      // ...
      return sb.ToString();
   }

   // ...

}
```

Entretanto, como o código de verificação desempenha um papel crucial na proteção do acesso aos seus documentos, nós
agora oferecemos a classe `AlphaCode` no pacote *Lacuna.RestPki.Client* para fazer a geração dos códigos.

<a name="update-code" />
## Atualizando sua aplicação para usar o *AlphaCode*

> [!WARNING]
> A lógica de geração de códigos de verificação passou por uma auditoria minuciosa e recebeu importantes melhorias.
> Por isso, recomendamos fortemente que você atualize a sua aplicação para utilizar a classe *AlphaCode* ao invés de utilizar
> o código-fonte antigo.

Você provavelmente copiou anteriormente os métodos (agora obsoletos) `GenerateVerificationCode`, `FormatVerificationCode` e `ParseVerificationCode`
dos exemplos para o seu código. Para atualizar a sua aplicação, simplesmente substitua a implementação desses métodos por chamadas aos métodos
da classe *AlphaCode*:

```cs
public static string GenerateVerificationCode() => AlphaCode.Generate();

public static string FormatVerificationCode(string code) => AlphaCode.Format(code);

public static string ParseVerificationCode(string formattedCode) => AlphaCode.Parse(code);
```

## Princípios de projeto

A classe *AlphaCode* gera códigos alfanuméricos feitos para serem lidos por pessoas seguindo os princípios abaixo:

1. Os códigos devem de fácil leitura
1. Os códigos devem poder ser facilmente digitados com baixo risco de confundir caracteres similares como `O` e `0`
1. Os códigos devem ter relativa alta entropia com relação ao tamanho do código (alto número de possíveis códigos relativo ao tamanho
   do código, permitindo ao desenvolvedor escolher um código de tamanho relativamente pequeno)

Para melhorar a legibilidade, são utilizadas apenas letras maiúsculas e não são utilizados os caracteres `O`, `0`, `1` e `I`. A entropia
por caractere é relativamente alta, se comparada com um código hexadecimal: como existem 32 possíveis caracteres, cada caractere contribui
com 5 bits à entropia total do código gerado (25% a mais do que num código hexadecimal), o que resulta em códigos menores para uma dada entropia mínima.

Por exemplo, para gerar um código com entropia de 80 bits (2^80 possíveis códigos), um código hexadecimal precisaria ter 20 caracteres,
enquanto que um código gerado pela classe *AlphaCode* só precisaria ter 16 caracteres.

De maneira geral, o uso da classe se dá da seguinte maneira:

* Durante a geração de uma versão para impressão de um documento assinado digitalmente:
  1. Chame `AlphaCode.Generate()` para obter um código, ex: `XXXXXXXXXXXXXXXX`
  1. Armazene o código no seu banco de dados indexando o documento ou entidade associada
  1. Chame `AlphaCode.Format(code)` para formatar o código de maneira mais legível, ex: `XXXX-XXXX-XXXX-XXXX`
  1. Escreva o código formatado no PDF de versão para impressão sendo gerado
* Durante a verificação de um documento:
  1. Solicite ao usuário o código de verificação
  1. Chame `AlphaCode.Parse(code)` para remover pontuações que o usuário possa ter digitado, obtendo de volta o código não-formatado
  1. Pesquisa no seu banco de dados o documento ou entidade pelo código não-formatado

## Veja também

* [Usando o Rest PKI em .NET](index.md)