# Como criar um chatbot para seus próprios arquivos usando a Azure OpenAI em 2024

Esse é um processo novo, e utiliza as funções "Preview" do Azure, talvez quando você for criar o seu já terão algumas diferenças. Mas não deixa de ser funcional.

Essa solução pode ser criada também diretamente usando as APIs do OPENAI, sem a necessidade de uma assinatura do Azure.

Mas confesso que a Azure facilita e muito o serviço. Além de já entregar um aplicativo de chatbot que autentica no seu AD (ou da sua empresa) e nasce pronto para ser customizado. Além de que, se você está começando pode usar um pouco dos seus créditos gratuitos sem pesar muito no bolso.

O passo a passo vai ser bem detalhado, quem ja manja das paradas pode pular algumas partes.


## Etapa 1: Autenticação e criação do recurso da OpenAI

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

No canto direito vai surgir a tela avisando que o recurso está sendo criado, é só aguardar.

![Captura de tela de 2024-04-16 18-29-35](https://github.com/pedropberger/tutorials/assets/98188778/d4c5019f-06b8-4f8b-b962-314efb23dae9)

.. e pronto, recurso funcionando. Agora é só alegria.

![Captura de tela de 2024-04-16 18-30-57](https://github.com/pedropberger/tutorials/assets/98188778/e4cc0205-4593-44b6-8f09-638283c93222)

# Etapa 2: Azure OpenAI Studio

Aqui começa o desenvolvimento do chatbot. Pra isso a Azure agora disponibiliza o Azure OpenAI Studio, que traz os seguintes benefícios:

- O Azure OpenAI Studio unifica capacidades de diferentes serviços de IA do Azure.
- Suporte simplificados a modelos como ChatGPT, DALL-E, Ada e outros no serviço Azure OpenAI.
- Várias opções de segurança, conformidade e preços.
- Ele permite que você crie, avalie e implante soluções geradas por IA.
- Seja para chatbots, assistentes virtuais ou outras aplicações, o estúdio oferece uma experiência integrada

Lá você controla quais modelos, como vai usá-los e até recebe exemplos de como consultar as API (mas isso é assunto para outro tutorial).

Bora lá. Para começar, procure pelo seu recurso recém criado e clique sobre ele:

![Captura de tela de 2024-04-16 18-31-42](https://github.com/pedropberger/tutorials/assets/98188778/fded6ea5-72cb-4e84-b250-c71620bf103e)

Você ira para uma tela similar a tela abaixo. Aqui você controla tudo que é gerado pelo seu recurso.

![Captura de tela de 2024-04-16 18-32-44](https://github.com/pedropberger/tutorials/assets/98188778/f0642943-88fa-421c-a7e5-e580b4061cfc)

Em especial, a aba Develop traz as chaves de utilização, região em que foi criado o recurso e o Endpoint. Esses parâmetros não são mutáveis, e são de grande importancia no deploy de aplicações ou consumo dos modelos via API. Na dúvida, lembre sempre como achá-los e saiba dos riscos de expor as Chaves.

![Captura de tela de 2024-04-16 18-32-52](https://github.com/pedropberger/tutorials/assets/98188778/4e537ca5-e685-420d-9287-dd604d6639e6)

Na aba Monitor você controla o uso e consumo dos seus modelos e aplicações. São diversas métricas com diferentes finalidades. Você consegue monitorar desde o uso do recurso, ou tokens (que impacta no preço) até tempo de uso de treinamento para o ajuste fino.

![Captura de tela de 2024-04-16 18-33-08](https://github.com/pedropberger/tutorials/assets/98188778/773fd484-03e3-41ef-b507-80fa1936aa93)

Agora clique em "Go to Azure OpenAI Studio" no canto superior esquerdo e vamos para o que realmente interessa.

![Captura de tela de 2024-04-22 16-08-19](https://github.com/pedropberger/tutorials/assets/98188778/cc74e5d5-3f9a-49be-a051-4a801fce60b3)

Você vai esbarrar com uma tela similar a essa:

![Captura de tela de 2024-04-22 16-36-18](https://github.com/pedropberger/tutorials/assets/98188778/78a249e9-aca0-46cf-aa8c-cab591032391)

Aí tem vários tutoriais prontos para você brincar e aprender. Mas o que aconselho a prestar atenção é nesse menu lateral e suas funcionalidades:

Playground
- Chat: O recurso de chat permite que você interaja com modelos de linguagem, como o ChatGPT, para criar experiências de aplicativos com base em conversas. Você pode enviar prompts e receber respostas geradas pelo modelo. É útil para criar assistentes virtuais, chatbots e outras aplicações de processamento de linguagem natural1.
- Completions: Completions referem-se a modelos de preenchimento automático, como o GPT-4. Esses modelos podem prever e gerar continuamente texto com base em um contexto inicial. Eles são úteis para tarefas como autocompletar frases, gerar código ou escrever histórias1.
- DALL-E: O DALL-E é um modelo de IA que gera imagens a partir de descrições de texto. Ele é capaz de criar arte visual com base em prompts de texto, permitindo que você explore a criatividade na geração de imagens1.
- Assistentes (Visualização): A funcionalidade de assistentes está em visualização e permite que você crie seus próprios assistentes virtuais personalizados usando modelos de IA, como o GPT-4 e o DALL-E. Você pode integrar seus próprios dados, experimentar com prompts e criar fluxos de diálogo personalizados para criar assistentes específicos para suas necessidades2.

Management
- Deployments: Os deployments são onde você implanta os modelos treinados para uso em produção. Você pode implantar modelos de linguagem, visão computacional e outros tipos de modelos para atender às necessidades do seu aplicativo.
- Models: Aqui você encontrará uma variedade de modelos de IA disponíveis para uso. Esses modelos incluem recursos como processamento de linguagem natural, visão computacional e muito mais. Você pode escolher o modelo adequado para sua aplicação específica.
- Data Files: Os arquivos de dados são usados para treinar e avaliar modelos. Você pode carregar seus próprios dados para treinamento ou usar conjuntos de dados pré-existentes para criar modelos personalizados.
- Quotas: As quotas se referem aos limites de uso para implantação e inferência de modelos. Cada modelo tem uma cota associada, e você pode ajustar essas cotas conforme necessário para atender às demandas do seu aplicativo.
- Content Filters: Os filtros de conteúdo são usados para garantir que as respostas geradas pelos modelos estejam dentro de limites aceitáveis. Isso é especialmente importante para evitar conteúdo inadequado ou ofensivo em aplicativos de produção.

Da vontade de começar no Playground né. Mas o primeiro passo é clicar em Deployment.

![Captura de tela de 2024-04-16 18-34-24](https://github.com/pedropberger/tutorials/assets/98188778/51919801-3d69-473f-a46f-5c641d9ef763)

E pra quê começar pelo Deployment? Simples, sem modelo você não tem IA. O chatbot ou o que você quiser fazer vai precisar de um modelo para se basear (e consumir recursos).

Gosto de enfatizar que o modelo é o motor carro. Seus créditos no Azure são o combustível. O Chatbot é só a lataria. Carro sem motor não anda.

Para criar um, clique em "Create new deployment" e surgira uma tela com várias opções:

![Captura de tela de 2024-04-22 16-37-35](https://github.com/pedropberger/tutorials/assets/98188778/1c7d631c-a7e3-40c7-b4d2-b06399facb63)

Entendendo a tela acima e definindo os parâmetros:

Select a model: Aqui você decide seu modelo. Vai ser uma IA Generativa? seleciona um GPT que te agrade. É imagem que você quer? DALL-E. Embbeding? Ada. Para manter a coerencia desse tutorial, selecione um GPT. Eu vou no mais barato, gpt-35-turbo. A ideia desse chatbot é só ler uma biblioteca de documentos e responder sobre ela. Não preciso de informações recentes ou capacidade de passar no vestibular. Mas cada um sabe do que precisa. Se quiser saber mais sobre qual modelo, [pesquisa nesse site que você encontra tudo](https://www.google.com).

Model version: Escolha a versão. Não da pra dizer como isso vai impactar. Escolha a que mais te agrade ou fique na padrão que não tem erro.

Deployment type: Standard, sempre Standard.

Deploymento name: Escolha um nome para o modelo. Você pode ter mais de um modelo do mesmo tipo, ou de tipos diferentes. O futuro a Deus pertence, então nomeie com sabedoria.

Content filter: Por hora não da para selecionar nada além do defalt.

Tokens per Minute Rate Limit: Esse é importante. Aqui você restringe quantos tokens você poderá consumir no máximo por minuto. Eu recomendo deixar o mínimo razoável que você acha que irá utilizar e habilitar a cota dinâmica. Mas se você vai implantar vários modelos é importante pensar nos limites antes. Atualmente por Subscription você tem o limite de 240k tokens por minuto, e essa cota vale PARA TODOS MODELOS, ou seja, se você tem outra aplicação que consome tokens, é importante limitar ela para não impactar no seu chatbot. Você pode visualizar as cotas e seus limites na aba Quotas.

Pronto. Daí é só fazer o deploy e ver a tela abaixo.

Dica importante: como todo recurso Azure, você pode criar e apagar quantas vezes quiser, mas as cotas do modelo demoram 48 horas para dar um "purge". Então se você sai criando e apagando modelos, fique atendo as cotas disponibilizadas para cada modelo, para não ter que esperar um tempão para elas voltarem. Isso aconteceu comigo e a solução eu [achei aqui](https://learn.microsoft.com/en-us/azure/ai-services/recover-purge-resources?tabs=azure-portal#purge-a-deleted-resource).

![Captura de tela de 2024-04-22 16-37-45](https://github.com/pedropberger/tutorials/assets/98188778/23840a85-a8f1-4c9d-80a5-03b91be3a508)

Deploy feito meus amigos, vamos criar o chatbot.

## Passo 3: Indexando os dados

Obviamente, clique na barra lateral no segundo item, o "Chat". E seja bem vindo ao Chat Playground. Aqui você já pode conversar, testar parâmetros e se familiarizar com seu chat que já tem como motor o modelo que você fez deploy acima.

![Captura de tela de 2024-04-22 16-38-03](https://github.com/pedropberger/tutorials/assets/98188778/cb2d218c-b88d-4c50-9843-309e7f0d9621)

Nosso objetivo é um chatbot que lê arquivos, então vamos direto ao ponto. Clique em Add your data. E vamos adicionar seus textos.

![Captura de tela de 2024-04-22 16-38-25](https://github.com/pedropberger/tutorials/assets/98188778/ada23fc3-62c1-4867-ab3d-12365cccf943)

Logo de cara você define o Data Source, e escolhe a opção de fazer upload dos seus arquivos. Você pode importar de outras opções, mas não vamos cobrir nesse tutorial.

![Captura de tela de 2024-04-22 16-38-50](https://github.com/pedropberger/tutorials/assets/98188778/57f191a6-9bae-4ce0-87e5-fed5c7fc77ed)

Em seguida você seleciona a Subscription, e depois seleciona o recurso de armazenamento Blob em que serão armazenados os arquivos. Se você não tem esse recurso é só clicar em "Create a new Azure Blob storage resource", seguir o fluxo e criar uma Storage Account. Lembrando que precisa estar na rede e região do recurso Azure OpenAI.

![Captura de tela de 2024-04-22 16-39-02](https://github.com/pedropberger/tutorials/assets/98188778/83731d18-458f-47e7-9fe7-8b6b8141a267)

O proximo passo é a AI Search. Aqui você vai criar os Indexadores e Índices.

Os Indexadores e Índices são componentes essenciais no Azure OpenAI On Your Data, que permite que os desenvolvedores conectem, ingeram e baseiem seus dados corporativos para criar copilotos personalizados rapidamente. Vamos entender o que são esses componentes:

Indexadores:
- Os indexadores são responsáveis por processar os dados que o serviço monitora. Quando todos os dados são processados, o OpenAI do Azure dispara o segundo indexador.
- Esse segundo indexador armazena os dados processados em um serviço da Pesquisa de IA do Azure.
- Os indexadores são fundamentais para a busca eficiente e a recuperação de informações relevantes a partir dos dados corporativos.

Índices:
- Os índices são estruturas de dados que organizam e otimizam o acesso aos dados.
- Eles permitem que o sistema execute consultas de maneira eficiente, acelerando a busca e a recuperação de informações.
- No contexto do Azure OpenAI, os índices são usados para melhorar a precisão das respostas do modelo e agilizar a interação com os dados.

Em resumo, os indexadores processam os dados e os armazenam em índices otimizados para facilitar a busca e a análise. Esses componentes trabalham juntos para permitir que o Azure OpenAI On Your Data ofereça respostas mais precisas e eficientes com base nos dados corporativos.

Clique para criar um Indexador e vá para a seguinte tela:

![Captura de tela de 2024-04-22 16-41-38](https://github.com/pedropberger/tutorials/assets/98188778/ea9a8ebd-4b8f-4dbf-905d-9079950fb898)

Aqui você seleciona suas opções e da um nome para o serviço de pesquisa. Selecione a região certa. É opcional (e recomendável) alterar o Pricing tier de acordo com a quantidade de dados que você planeja indexar. Porém esteja ciente que a opção "Free" não funcionará com o chatbot. Pelo menos não hoje. Na primeira vez que fiz, só aceitava a Standard que era cara pra caramba. Agora já da pra fazer com opções menos onerosas.

![Captura de tela de 2024-04-22 16-41-55](https://github.com/pedropberger/tutorials/assets/98188778/6d5bf62c-53fe-4102-8751-d9d5bef9d886)

Defina a quantidade de réplicas e partições. Se você não entende ou não precisa de escalabilidade, pule essa parte, ajuste a Rede, as Tags e seja feliz após o Review+Create

![Captura de tela de 2024-04-22 16-42-25](https://github.com/pedropberger/tutorials/assets/98188778/abe99d57-7f94-4e39-83bb-7109d43fd0e2)

Se a tela abaixo aparecer, clique em Turn on CORS para continuar. A explicação está no warning.

![Captura de tela de 2024-04-22 16-40-55](https://github.com/pedropberger/tutorials/assets/98188778/1f9d32da-49ac-440c-9106-88a5871c5651)

Coloque um nome para o Índice e depois é só criar seguir o baile.

![Captura de tela de 2024-04-22 16-56-12](https://github.com/pedropberger/tutorials/assets/98188778/c0cf0ed9-1338-498a-be8d-9052913e62b3)

Na tela em sequência você sobe seus arquivos. É basicamente um Drag and drop. Só aceita arquivos de texto nos formatos apresentados. Faça o upload e dê sequência.

![Captura de tela de 2024-04-22 16-56-25](https://github.com/pedropberger/tutorials/assets/98188778/8ad16fab-3456-4a52-bdef-a75ad862d800)

Na tela abaixo você administra como seus dados serão usados. Não da para selecionar nada além de Keyword. Porém você pode alterar o Chunk Size.

Da para escrever uma dissertação de mestrado sobre Chunk Size. Na minha experiência, deixo as seguintes dicas:

- Você tem arquivos pequenos e de poucas páginas: recomendo size de 256 ou 512.
- As respostas serão relacionadas a trechos pequenos dos documentos: 256 ou 512.
- Arquivos de texto grandes: 1024 ou 1536.
- Quer respostas que incorporem diversas partes do texto ou resumam páginas inteiras: 1024 ou 1536.

Na dúvida vai em 1024 que dá bom também.

É engraçado que existe muita subjetividade, e as melhores referencias sobre o tema dizem que você deve ajustar conforme o seu "feeling" e ir refinando com base respostas que você tem recebido e no feedback dos usuários. Não temos ainda funções de otimização ou que automatizem o melhor ajuste.

![Captura de tela de 2024-04-22 17-00-27](https://github.com/pedropberger/tutorials/assets/98188778/4b468949-f24e-4ce7-8367-feab35ec46e8)

De sequência e aí é só aguardar o processo de ingestão dos seus arquivos. Eles serão pré-processados e indexados um a um. Se der algum problema, é só refazer esse passo. Os problemas mais comuns são arquivos em formatos não compatíveis, índice já existente de outros deploys e problemas para acessar sua Storage Account.

![Captura de tela de 2024-04-22 17-00-40](https://github.com/pedropberger/tutorials/assets/98188778/e0e0480d-64c9-4a23-9e29-0c4447f04e06)

E pronto, agora só falta refinar os parâmetros.

# Etapa 4: O Chatbot

Agora já temos arquivos indexados e vinculados a nosso recurso Azure OpenAI. Chegou a hora de ajustar os detalhes sobre o prompt e os parâmetros. A qualquer momento durante essa etapa você já pode ir testando o chatbot na tela principal, que ele dará as respostas conforme os parâmetros ajustados em tempo real.

O primeiro passo antes de tudo é fazer os ajustes em como seu chatbot usará os dados. Para isso, clique em "Advanced settings" conforme na tela abaixo e serão apresentadas 3 opções.

![Captura de tela de 2024-04-22 17-18-29](https://github.com/pedropberger/tutorials/assets/98188778/b4dfc65e-e018-4f64-be6d-4083b300663c)

A primeira caixa de seleção é onde você define se o chatbot poderá responder questões de qualquer tópico (um ChatGTP raiz) além das relacionadas aos documentos. Eu sugiro fortemente em deixar ela marcada se seu interesse é de fato criar um chatbot que responda perguntas sobre os documentos indexados. Existem parâmetros para limitar o quanto ele pode sair do escopo dos documentos, mas todos meus testes com usuários onde não deixei essa caixinha marcada foram frustrantes. Usuários dificilmente seguem um manual e as vezes até se es

O segundo passo é definir a "role" do seu chatbot. Vem algumas pré configuradas para você visualizar e testar. Mas como isso vai para produção, você deve setar como Default (ou deixar em branco) e setar a System Message, conforme na tela abaixo. Na System Message você diz para seu chatbot o que ele é e como deve responder. Quem já estudou o básico do básico do prompt conhece a clássica frase "Fale como um especialista" antes de conversar com o GPT. Aqui a função vai ser semelhante, mas você diz em que ele será especialista. No nosso caso colocamos "Fale como um funcionário do RH. Sempre responda em Português Brasileiro". Depois de inserir a informação é só clicar em salvar.

![Captura de tela de 2024-04-22 17-18-21](https://github.com/pedropberger/tutorials/assets/98188778/7b80b2eb-225a-4ebd-a2a6-83cf7a4930a2)



![Captura de tela de 2024-04-22 17-18-41](https://github.com/pedropberger/tutorials/assets/98188778/99d20512-a980-4d81-8ab7-67cdd7ea0dc1)

![Captura de tela de 2024-04-22 17-18-51](https://github.com/pedropberger/tutorials/assets/98188778/876eea6a-140e-488a-83a4-fc424d4ed065)

![Captura de tela de 2024-04-22 17-19-19](https://github.com/pedropberger/tutorials/assets/98188778/fe162f61-f757-4bee-8113-919ca7e3f562)

![Captura de tela de 2024-04-22 17-20-17](https://github.com/pedropberger/tutorials/assets/98188778/8bd333f6-d41c-4726-bd9a-623d73b469ec)

![Captura de tela de 2024-04-22 17-26-05](https://github.com/pedropberger/tutorials/assets/98188778/d11bee24-2132-4651-b6da-184d26194fd1)

![Captura de tela de 2024-04-22 17-42-16](https://github.com/pedropberger/tutorials/assets/98188778/c427a7d1-7e8c-4bf5-8d88-a16bc264f058)


