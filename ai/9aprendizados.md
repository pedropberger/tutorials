# 9 lições aprendidas após um 9 meses implantando agentes de IA em prod

A um mês atrás foi o lançamento do programa AMPL.IA no MPES, fruto dos últimos 9 meses de trabalho, que foram uma mudança bem interessante de direcionamento na minha carreira.

Esse ultimo ano eu estive dedicado quase que exclusivamente em todas as etapas do desenvolvimento de agentes de IA, da arquitetura dos serviços, passando pela prototipagem e chegando até a infraestrutura de produção. Saí da zona de conforto dos pipelines de dados, e aprendi demais. Quando a gente olha pra trás, um caminho curvo às vezes parece uma reta, mas na verdade foram muitas derrapadas durante todo o processo. Queria aproveitar para compartilhar 9 aprendizados que tive (a partir de umas 90 derrapadas), e espero que ajude quem está querendo se aventurar nesse mundo novo da criação de Agentes de IA com IA Generativa.

1 - Foque nas dores

É importante entender o que realmente afeta o usuário. IA parece mágica e o povo costuma chamar atenção com firula, mas só o que elimina as dores é realmente útil, irá permanecer e ganhar adesão. E acredite, o stakeholder às vezes não conhece as dores do usuário, e às vezes nem o próprio usuário. Por isso dialogue, explique, revise conceitos, converse de novo e, principalmente, faça protótipos para eles brincarem.

2 - Faça protótipos

Agora, pegando o gancho da dica anterior, faça protótipos. Lembro que o maior desafio que tive foi em uma das soluções em que apenas segui rigorosamente os requisitos técnicos e de negócio. Acho que todo mundo já passou por isso (esse artigo fala de 9 aprendizados, mas poderiam ser 9 grandes erros que cometi). Resultado: deu ruim.

Sem querer ser repetitivo, mas já sendo: as pessoas confundem IA com magia e às vezes sabem menos sobre o processo (e sobre si mesmas) do que pensam. Prototipar hoje é fácil e é o melhor uso (e o único que considero seguro atualmente) do *vibe coding*. Não exige escala, frontends bonitos ou UX/UI bem projetados. Crie, deixe a galera testar, colha o feedback, faça testes A/B, grupos focais e, assim, conheça melhor o seu cliente para no fim entregar algo realmente útil para ele.

3 - Modularização

IA está na moda, então as pessoas querem integrar IA em seus sistemas. O ponto é que muitos stakeholders não técnicos (e, infelizmente, às vezes técnicos) acham que um LLM é um software que você espeta com um pendrive, instala nos sistemas da casa e, bum, está tudo resolvido. Mas a verdade é que é uma solução de múltiplas camadas: prompt, contexto, segurança, bancos de dados, e que frequentemente se envolve com outros sistemas (OCR, Docker, etc.). Isso tudo antes de se integrar ao sistema principal. O que aprendi com isso tudo é: não tente integrar a alma do seu agente ao seu sistema, modularize tudo. Os ganhos em manutenção, operação, segurança e escala serão imensuráveis.

4 - Quebre seus agentes em agentes menores

Essa daria para virar um artigo técnico inteiro (fica pro futuro), então vou me limitar a contar a história: com as janelas de contexto gigantes que temos hoje, juntando com LLMs competentes, nossa tendência é juntar todas as informações que temos, temperar com um prompt afiado e esperar que todos os problemas se resolvam. E talvez, na maioria das vezes, vá resolver. O problema é que "na maioria das vezes" está longe do ótimo e do esperado para um sistema confiável.

Reduza seu contexto (vai reduzir alucinações), quebre seu processo em etapas, construa agentes especializados, construa agentes validadores, dê a cada um um serviço específico e acredite: sua taxa de erros vai cair muito, seu ruído vai diminuir (Kahneman fala disso para humanos, mas hoje também é um viés em IA), e você vai criar inúmeras oportunidades de otimização. Complica um pouco? Sim, vai ter que orquestrar, mas vale a pena.

5 - Engenharia de Contexto > Engenharia de Prompt

Vou repetir, isso deveria virar um mantra: Engenharia de Contexto > Engenharia de Prompt, Engenharia de Contexto > Engenharia de Prompt, Engenharia de Contexto > Engenharia de Prompt. Ainda mais por conta dos milhares de gurus do prompt que estão surgindo mundo afora. Não estou desmerecendo um prompt bem feito (que sim, faz milagres), mas ta longe de ser algo muito complexo e mais longe ainda de ser o fator mais importante na criação de um agente.

Hoje, os LLMs são muito competentes para entender perguntas e escrever textos. O prompt é a cobertura do bolo, ele dá a forma à informação. Mas o contexto é o que dará o ouro ao agente. Assim como a qualidade dos dados é o principal para um bom modelo (DL ou ML) ou relatório, o contexto vai ditar a qualidade da resposta do seu agente. Quando colocamos algo em produção, isso fica mais nítido ainda. Um bom contexto reduz alucinações, economiza tokens e melhora a experiência do usuário como um todo. *É o combustível de alta qualidade para os seus agentes especializados.*

6 - Frameworks podem ajudar, mas ainda são opcionais

Quando conheci o LangChain, foi amor à primeira vista. O framework fazia tudo que eu queria e parecia que já haviam implementado a solução de todos os meus problemas como funções simples e objetivas. Confesso que já fazem dois anos que comecei essa brincadeira e foi um grande aprendizado, mas o ponto é que, no fim, nunca precisei utilizar em produção. Achava que o LangChain estava para os LLMs como o pandas e o polars estão para a manipulação de dados. Ledo engano, não era amor, era cilada.

Ainda na construção dos primeiros protótipos e testes, tive problemas com versões de bibliotecas e clientes. Acabei reescrevendo tudo na mão, com minhas próprias funções. Além de ter entendido melhor tudo que acontecia, ganhei velocidade e eficiência. [tenho um texto chorando isso] O aprendizado que fica é que não, você não é dependente de um framework para implantar agentes em produção.

7 - Escala é um desafio constante

Falar de escala, vamos lá. Escala foi meu primeiro soco na cara.

Parece simples, mas não é. Na semana de deploy em prod dos primeiros agentes, tive uma parada total, não minha, do agentes, todos. Erro meu, ficou o aprendizado. Estimei uma demanda muito maior que o piloto (o equivalente ao triplo de usuários possíveis), mas não previ tão bem o comportamento do usuário e a capacidade da solução de OCR em consumir recursos. E, erroneamente, tanto as APIs que geravam a fila quanto os Workers estavam consumindo processamento do mesmo cluster. A competição pendeu pro OCR, que topou a cpu em 100% e não saiu. Até reiniciar os containers e movimentar as cargas, a fila cresceu, cresceu e cresceu sem parar.

Outro ponto foram os bancos de dados.

Os agentes que desenvolvemos faziam um trabalho bonito de busca de contexto, incluindo metadados de processos e documentos que estavam armazenados em SQL. Sempre tive em mente que um bom banco tem que dar conta de milhares de consultas por minuto, e isso é um fato. Mas fazer isso de forma rápida, sempre, é outra questão, e que se resolve de uma forma bem diferente que um cientista de dados precisa resolver. Quando precisa gravar algo então, é bom ter bem mentalizado o conceito de atomicidade. Enfim. Entenda suas querys, otimize-as e, se necessário, peça apoio do seu DBA para garantir que sejam criados índices que façam sentido.

O aprendizado aqui é que rodar para um é simples; para mil, a coisa deixa de ser trivial e exige profissionalismo e boa arquitetura.

8 - Monitore tudo

Logs são essenciais. Ponto. E quanto mais, melhor. Você nunca sabe quando vai precisar. Tive um agente que escrevia uma peça após N processos de diferentes agentes. Um dia, um usuário reportou uma alucinação (a primeira a gente nunca esquece). Graças aos logs, consegui identificar qual parte do processo, qual subagente e qual modelo utilizado de fato vacilou e, por fim, corrigir com uma linha de código.

Quando você orquestra agentes, erros podem se perpetuar muito rapidamente. Mesmo que você tenha *guardrails* e validadores, é sempre bom saber a origem do problema para corrigir rapidamente, e para isso você só pode recorrer aos logs. Recomendo salvar (temporariamente) todos os prompts, respostas, contextos utilizados, metadados e interações. De bônus, você irá utilizar esses dados para avaliar seus modelos e operações com o que gostar mais e precisar: RAGAS, LLM-as-a-Judge & G-EVAL agradecem.

9 - Faça amizade com o time de desenvolvimento, infra e DevOps 

Depois de toda a jornada técnica, a lição mais importante é sobre pessoas.

Eis a dica mais recompensadora, na minha opinião. Ninguém é uma ilha. Tenho percebido que os times de IA surgidos nos últimos anos geralmente nasceram como desmembramentos de times de dados ou de dev, e mais raramente como uma geração espontânea. Independentemente do lado que vieram, tiveram que aprender algo novo.

Deixo claro que não estou falando de desenvolvimento de modelos ou *fine-tuning*. Estou falando de implantar agentes de IA em produção, integrados a sistemas consolidados. O ponto é: ao fazer isso, você e seu time vão precisar de apoio, sempre. E acho que esse foi o meu maior ganho nesses 9 meses. Pude interagir e aprender muito com gênios sempre prontos para me mostrar atalhos que eu achava inviáveis, enxergar falhas de segurança invisíveis, entender conceitos que eu achava que estavam muito além da minha alçada. E, de bônus, me diverti muito e dei muita risada.

Fica aqui meu agradecimento a toda a equipe.
