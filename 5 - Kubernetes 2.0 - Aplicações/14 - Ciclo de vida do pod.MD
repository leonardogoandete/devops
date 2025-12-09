## Ciclo de Vida do Pod

O ciclo de vida de um Pod no Kubernetes é gerenciado pelo **kubelet**, que é o agente responsável por garantir que os contêineres dentro do Pod estejam em execução conforme especificado. O ciclo de vida do Pod pode ser dividido em várias fases principais:
1. **Pending**: O Pod foi aceito pelo sistema, mas um ou mais dos seus contêineres ainda não foram criados. Isso pode ocorrer devido a várias razões, como a falta de recursos no nó ou a espera por volumes persistentes.
2. **Running**: Todos os contêineres do Pod foram criados com sucesso e pelo menos um deles está em execução. O Pod está ativo e funcionando.
3. **Succeeded**: Todos os contêineres do Pod terminaram sua execução com sucesso (código de saída 0). Este estado é comum para Pods que executam tarefas de curta duração.
4. **Failed**: Um ou mais contêineres do Pod terminaram sua execução com falha (código de saída diferente de 0). O Pod não será reiniciado automaticamente, a menos que seja configurado para isso.
5. **Unknown**: O estado do Pod não pode ser determinado, geralmente devido a problemas de comunicação com o nó onde o Pod está em execução.