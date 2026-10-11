# Meu Brasil

Mapa de municípios brasileiros para registrar cidades visitadas. Ele funciona imediatamente com backup no navegador e, depois de configurado o Firebase, usa login Google e sincronização em tempo real entre navegadores.

## O que esta versão faz

- Marca e desmarca municípios no mapa; mantém exportação e importação de backup JSON.
- Mantém as marcações antigas no `localStorage` (`meu-brasil-visitas-v1`).
- Ao entrar com Google, envia as marcações locais para a conta sem apagar o que já estava na nuvem.
- Atualiza a lista e o mapa automaticamente quando a mesma conta altera visitas em outro navegador.
- Se a nuvem estiver indisponível, conserva o backup local e avisa a pessoa usuária.

Os dados na nuvem ficam em `users/{uid}/visits/{cidade}`. Cada documento pertence a uma única conta e a regra fornecida bloqueia acesso por outras contas.

Ao importar um backup com login ativo, o conjunto de cidades da conta é substituído pelo conjunto do arquivo. Se a consulta à nuvem falhar durante essa operação, o backup permanece salvo no navegador, mas é preciso importá-lo novamente quando a conexão voltar para remover da conta as cidades que não estão no arquivo.

## Configurar Firebase

Você precisa criar ou usar um projeto Firebase próprio; não existe credencial de projeto no repositório.

1. No [Firebase Console](https://console.firebase.google.com/), crie um projeto ou abra um existente e registre um **app Web** (`</>`).
2. Copie o objeto `firebaseConfig` que o Console mostrar. Em `index.html`, mantenha os nomes das seis propriedades e substitua cada valor que começa com `COLE_AQUI_` pelo valor correspondente do objeto fornecido. A configuração web (incluindo `apiKey`) pode ficar no HTML; ela não concede acesso aos dados. **Nunca** envie chaves de service account, segredos OAuth ou outras chaves privadas ao repositório.
3. Em **Authentication → Sign-in method**, habilite **Google**. Em **Authentication → Settings → Authorized domains**, adicione `gabrieldalosse-star.github.io`. Adicione também um domínio personalizado, se houver, e `localhost` para testar localmente.
4. Preserve o `authDomain` entregue pelo Firebase (normalmente `seu-projeto.firebaseapp.com`). Não o troque por um domínio `github.io`: o handler de login continua hospedado pelo Firebase.
5. Em **Firestore Database**, crie o banco no modo de produção. Em **Rules**, copie todo o conteúdo de [`firestore.rules`](./firestore.rules) e clique em **Publish**. Não deixe o banco em *test mode*.
6. Recarregue o site publicado, marque uma cidade antes de entrar e clique em **Entrar com Google**. A mensagem deve mudar para “Sincronizado na conta Google”.

O app usa os módulos Firebase Web `13.0.0` diretamente do CDN. O login é iniciado por clique e usa popup, que é a opção mais estável para GitHub Pages.

## Publicar no GitHub Pages

1. Envie `index.html`, `README.md` e `firestore.rules` para a branch `main`.
2. No repositório, abra **Settings → Pages**. Em **Build and deployment**, selecione **Deploy from a branch**, escolha `main` e a pasta `/(root)`, depois clique em **Save**.
3. Aguarde a conclusão do deploy. Para a configuração padrão deste repositório, a URL esperada é `https://gabrieldalosse-star.github.io/meu-brasil/`.
4. Abra a URL por HTTPS e teste o login. Se aparecer `auth/unauthorized-domain`, confira se `gabrieldalosse-star.github.io` foi adicionado em Authorized domains (somente o host, sem `https://` e sem `/meu-brasil/`).

Para testar localmente, use um servidor HTTP (não abra o arquivo via `file://`), por exemplo uma extensão de servidor estático do editor. O login Google exige um endereço HTTP/HTTPS válido.

## Checklist de validação

1. Sem login, marque cidades, recarregue a página e confirme que elas continuam marcadas.
2. Entre com Google e confirme que as marcações locais aparecem na conta.
3. Em outro navegador ou janela anônima, entre na mesma conta e confirme que as visitas aparecem.
4. Marque uma cidade em um navegador e confirme a atualização automática no outro.
5. Entre com outra conta Google: ela deve começar sem ver as visitas da primeira conta.
6. No simulador de regras, confirme que acesso sem login é negado e que o UID da conta A não consegue ler, criar, alterar ou apagar dados em `users/{uid-da-conta-B}/visits/...`.

## Referências

- [Configurar Firebase para Web](https://firebase.google.com/docs/web/setup)
- [Login Google com Firebase](https://firebase.google.com/docs/auth/web/google-signin)
- [Escutar mudanças do Firestore em tempo real](https://firebase.google.com/docs/firestore/query-data/listen)
- [Regras de segurança do Firestore](https://firebase.google.com/docs/firestore/security/rules-fields)
