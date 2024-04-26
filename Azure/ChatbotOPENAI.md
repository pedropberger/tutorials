# Como criar um chatbot para seus próprios arquivos usando a Azure OpenAI em 2024

Esse é um processo novo, e utiliza as funções "Preview" do Azure, talvez quando você for criar o seu já terão algumas diferenças. Mas não deixa de ser funcional.

Essa solução pode ser criada também diretamente usando as APIs do OPENAI, sem a necessidade de uma assinatura do Azure.

Mas confesso que a Azure facilita e muito o serviço. Além de já entregar um aplicativo de chatbot que autentica no seu AD (ou da sua empresa) e nasce pronto para ser customizado. Além de que, se você está começando pode usar um pouco dos seus créditos gratuitos sem pesar muito no bolso.

O passo a passo vai ser bem detalhado, quem ja manja das paradas pode pular algumas partes.


## Passo 1: Autenticação e criação do recurso da OpenAI

Primeiro, entre no Portal Azure. Isso é igual para todos.

![Captura de tela de 2024-04-16 18-20-03](https://github.com/pedropberger/tutorials/assets/98188778/5622f5b4-59b5-4f05-a4c7-fa3d749ea1da)

Faça o login com suas credenciais (pessoais ou da sua empresa)

![Captura de tela de 2024-04-16 18-20-16](https://github.com/pedropberger/tutorials/assets/98188778/bb284728-0370-4625-80a2-8d40ce0eaa1a)

Usualmente, você vai ser essa barra. Clique no "+" e procure o recurso, ou apenas busque por "OpenAI" na barra superior.

![Captura de tela de 2024-04-16 18-21-32](https://github.com/pedropberger/tutorials/assets/98188778/13c84f15-cb96-4631-817c-3e667ea13768)

Cliquei no ícone da "Azure OpenAI" para entrar nas opções do recurso.

![Captura de tela de 2024-04-16 18-22-30](https://github.com/pedropberger/tutorials/assets/98188778/a506b056-e261-468c-b569-42c98c74b329)

Nessa tela você gerencia todas as opções sobre o recurso, inclusive opções sobre implantações de modelos que você irá fazer no futuro (GPT 3.5 turbo, GPT 4, DALL-e, etc).

Você pode criar um ou mais recursos OpenAI. A operação de criar o recurso não tem um custo, e pelo meus testes. Então você pode criar um recurso OpenAI para cada aplicação e chatbot, ou pode utilizar apenas uma para todas suas soluções. Isso vai depender da quantidade de serviços e da forma de sua equipe/organização trabalhar. Para criar o recurso é só clicar em "Create" no canto superior esquerdo.

![Captura de tela de 2024-04-16 18-23-03](https://github.com/pedropberger/tutorials/assets/98188778/9e8361d2-1c63-4604-bc5a-ebbbe923b840)

Você chegará na tela abaixo. Aqui começam as definições importantes que vão seguir para sempre com o recurso. Algumas podem ser modificadas depois do Deploy, outras não. Mas não precisa se preocupar muito com errar se for sua primeira vez, se der ruim é só apagar tudo e criar de novo.

Selecione sua Subscription e seu Resource group. Se você não conseguir, procure alguém que possa te dar permissão. As vezes não basta ter acesso ao Resource Group, você precisa de permissão para criar o recurso dentro daquele grupo de recursos.

Selecione a região. Dica importante sobre a região: Você precisará criar outros recursos nesse tutorial ou na sua vida, como uma Storage Account e um Serviço de Pesquisa, é interessante que estejam na mesma rede que o recurso da OpenAI, e para isso as vezes é necessário que sejam criados na mesma região, e nem todas regiões possuem todos recursos. Então aconselho escolher uma região e criar todos seus recursos e rede nela. E escolher uma região que possua todos recursos. Foi mal se soou prolixo.

Algumas regiões (como Brazil-South, por exemplo) ainda não possuem certos modelos que serão necessários nesse tutorial.

![Captura de tela de 2024-04-16 18-26-01](https://github.com/pedropberger/tutorials/assets/98188778/2f459747-93fa-455d-a8f9-5bf81df954ca)

Aqui na rede recomendo você pode criar ou utilizar uma rede virtual e subrede para sua segurança. Lembrando que todos os recursos precisarão ser criados na mesma rede.

Você também pode utilizar a primeira opção e liberar para todas as redes, incluindo a internet, para acessar o recurso. É arriscado? é. Mas para alguém consumir o seu recurso precisará da suas Chaves que serão criadas no futuro. Então na minha opinião de não especialista em segurança de TI, isso é um risco controlado, já que será um ambiente de desenvolvimento.

Vai fazer um deploy em produção? use uma rede fechada e consulte o administrador de redes do seu time.

![Captura de tela de 2024-04-16 18-26-34](https://github.com/pedropberger/tutorials/assets/98188778/2f28cb2d-4f2d-4f23-b988-f6255bc58439)

TAGs!

Aqui você coloca as TAGs

O que são tags?
- As tags são pares de chave-valor que você pode associar a recursos no Azure.
- Cada recurso ou grupo de recursos pode ter até 15 tags.
- Exemplo de formato de tag: Departamento: TI, Status: Teste, CentroDeCusto: RH.

Benefícios do uso de tags:
- Organização: Ajuda a organizar e categorizar recursos.
- Gestão: Facilita a busca e filtragem de recursos com base em critérios específicos.
- Acompanhamento de custos: Permite rastrear os custos associados a projetos ou departamentos específicos.

Veja como seu time uma forma organizada de usá-las. É algo opcional, cada equipe tem suas políticas e fique tranquilo, da para alterar/acrescentar novas depois.

![Captura de tela de 2024-04-16 18-27-40](https://github.com/pedropberger/tutorials/assets/98188778/a2ab9113-9f34-4057-9a8d-0c2b36ec788a)

Na tela final você revisa se tem alguma pendência e é só mandar bala, CREATE.

![Captura de tela de 2024-04-16 18-29-21](https://github.com/pedropberger/tutorials/assets/98188778/d6369ef6-11e3-4855-9dda-ab08d689be7d)

![Captura de tela de 2024-04-16 18-29-35](https://github.com/pedropberger/tutorials/assets/98188778/d4c5019f-06b8-4f8b-b962-314efb23dae9)

![Captura de tela de 2024-04-16 18-30-57](https://github.com/pedropberger/tutorials/assets/98188778/e4cc0205-4593-44b6-8f09-638283c93222)

![Captura de tela de 2024-04-16 18-31-33](https://github.com/pedropberger/tutorials/assets/98188778/ca59ee11-8c10-4acd-8f1d-c1e157ce3a81)

![Captura de tela de 2024-04-16 18-31-42](https://github.com/pedropberger/tutorials/assets/98188778/fded6ea5-72cb-4e84-b250-c71620bf103e)

![Captura de tela de 2024-04-16 18-32-44](https://github.com/pedropberger/tutorials/assets/98188778/f0642943-88fa-421c-a7e5-e580b4061cfc)

![Captura de tela de 2024-04-16 18-32-52](https://github.com/pedropberger/tutorials/assets/98188778/4e537ca5-e685-420d-9287-dd604d6639e6)

![Captura de tela de 2024-04-16 18-33-08](https://github.com/pedropberger/tutorials/assets/98188778/773fd484-03e3-41ef-b507-80fa1936aa93)

![Captura de tela de 2024-04-16 18-34-24](https://github.com/pedropberger/tutorials/assets/98188778/51919801-3d69-473f-a46f-5c641d9ef763)

![Captura de tela de 2024-04-22 16-08-19](https://github.com/pedropberger/tutorials/assets/98188778/cc74e5d5-3f9a-49be-a051-4a801fce60b3)

![Captura de tela de 2024-04-22 16-36-18](https://github.com/pedropberger/tutorials/assets/98188778/78a249e9-aca0-46cf-aa8c-cab591032391)

![Captura de tela de 2024-04-22 16-36-42](https://github.com/pedropberger/tutorials/assets/98188778/8cbaa123-257e-407e-999c-457bb4a2465f)

![Captura de tela de 2024-04-22 16-36-57](https://github.com/pedropberger/tutorials/assets/98188778/d231674c-8200-4929-a730-01415da9c5db)

![Captura de tela de 2024-04-22 16-37-14](https://github.com/pedropberger/tutorials/assets/98188778/6cf263d6-bce0-48fa-8320-b38c80bf501f)

![Captura de tela de 2024-04-22 16-37-35](https://github.com/pedropberger/tutorials/assets/98188778/1c7d631c-a7e3-40c7-b4d2-b06399facb63)

![Captura de tela de 2024-04-22 16-37-45](https://github.com/pedropberger/tutorials/assets/98188778/23840a85-a8f1-4c9d-80a5-03b91be3a508)

![Captura de tela de 2024-04-22 16-38-03](https://github.com/pedropberger/tutorials/assets/98188778/cb2d218c-b88d-4c50-9843-309e7f0d9621)

![Captura de tela de 2024-04-22 16-38-25](https://github.com/pedropberger/tutorials/assets/98188778/ada23fc3-62c1-4867-ab3d-12365cccf943)

![Captura de tela de 2024-04-22 16-38-50](https://github.com/pedropberger/tutorials/assets/98188778/57f191a6-9bae-4ce0-87e5-fed5c7fc77ed)

![Captura de tela de 2024-04-22 16-39-02](https://github.com/pedropberger/tutorials/assets/98188778/83731d18-458f-47e7-9fe7-8b6b8141a267)

![Captura de tela de 2024-04-22 16-40-55](https://github.com/pedropberger/tutorials/assets/98188778/1f9d32da-49ac-440c-9106-88a5871c5651)

![Captura de tela de 2024-04-22 16-41-38](https://github.com/pedropberger/tutorials/assets/98188778/ea9a8ebd-4b8f-4dbf-905d-9079950fb898)

![Captura de tela de 2024-04-22 16-41-55](https://github.com/pedropberger/tutorials/assets/98188778/6d5bf62c-53fe-4102-8751-d9d5bef9d886)

![Captura de tela de 2024-04-22 16-42-25](https://github.com/pedropberger/tutorials/assets/98188778/abe99d57-7f94-4e39-83bb-7109d43fd0e2)

![Captura de tela de 2024-04-22 16-42-39](https://github.com/pedropberger/tutorials/assets/98188778/ad452f9c-6b40-44f5-b0ff-6e88b2f8c892)

![Captura de tela de 2024-04-22 16-42-53](https://github.com/pedropberger/tutorials/assets/98188778/2e9e95c6-92ed-45d8-afd8-54cc63b63de0)

![Captura de tela de 2024-04-22 16-56-12](https://github.com/pedropberger/tutorials/assets/98188778/c0cf0ed9-1338-498a-be8d-9052913e62b3)

![Captura de tela de 2024-04-22 16-56-25](https://github.com/pedropberger/tutorials/assets/98188778/8ad16fab-3456-4a52-bdef-a75ad862d800)

![Captura de tela de 2024-04-22 17-00-27](https://github.com/pedropberger/tutorials/assets/98188778/4b468949-f24e-4ce7-8367-feab35ec46e8)

![Captura de tela de 2024-04-22 17-00-40](https://github.com/pedropberger/tutorials/assets/98188778/e0e0480d-64c9-4a23-9e29-0c4447f04e06)

![Captura de tela de 2024-04-22 17-18-21](https://github.com/pedropberger/tutorials/assets/98188778/7b80b2eb-225a-4ebd-a2a6-83cf7a4930a2)

![Captura de tela de 2024-04-22 17-18-29](https://github.com/pedropberger/tutorials/assets/98188778/b4dfc65e-e018-4f64-be6d-4083b300663c)

![Captura de tela de 2024-04-22 17-18-41](https://github.com/pedropberger/tutorials/assets/98188778/99d20512-a980-4d81-8ab7-67cdd7ea0dc1)

![Captura de tela de 2024-04-22 17-18-51](https://github.com/pedropberger/tutorials/assets/98188778/876eea6a-140e-488a-83a4-fc424d4ed065)

![Captura de tela de 2024-04-22 17-19-19](https://github.com/pedropberger/tutorials/assets/98188778/fe162f61-f757-4bee-8113-919ca7e3f562)

![Captura de tela de 2024-04-22 17-20-17](https://github.com/pedropberger/tutorials/assets/98188778/8bd333f6-d41c-4726-bd9a-623d73b469ec)

![Captura de tela de 2024-04-22 17-26-05](https://github.com/pedropberger/tutorials/assets/98188778/d11bee24-2132-4651-b6da-184d26194fd1)

![Captura de tela de 2024-04-22 17-42-16](https://github.com/pedropberger/tutorials/assets/98188778/c427a7d1-7e8c-4bf5-8d88-a16bc264f058)


