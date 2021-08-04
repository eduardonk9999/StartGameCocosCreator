# StartGameCocosCreator
Doc traduzida do projeto de start do Cocos Creator

######

Vamos dar uma olhada na função desses códigos. Primeiro, podemos ver um método cc.Class () global, o que é cc? cc é a abreviatura de Cocos, o principal namespace do mecanismo Cocos. E todas as classes, funções, propriedades e constantes no código do mecanismo são definidas neste namespace. E Class () é um método no módulo cc, que é usado para declarar classes no Cocos Creator. Para facilitar a diferenciação, chamamos a classe declarada com cc.Class chamada CCClass. O parâmetro do método Class () é um objeto de protótipo, a classe necessária pode ser criada definindo o parâmetro de tipo desejado na forma de um par de valores-chave no objeto de protótipo.
 


var Sprite = cc.Class({
    name: "sprite"
})

O código acima cria um tipo com o método cc.Class () e o atribui à variável Sprite. O nome da classe também é definido como sprite. Nomes de classes são usados ​​para serialização, que geralmente podem ser omitidos.

Para obter informações detalhadas sobre cc.Class, consulte Declarar classe com cc.Class.

Agora, voltamos ao editor de código e examinamos o código, que é a estrutura necessária para escrever um script de componente. Scripts com essa estrutura são os componentes no Cocos Creator, que podem ser montados em nós na cena e controlar várias funções do nó. Vamos começar definindo algumas propriedades e depois ver como podemos ajustá-las na cena.

Encontre a seção de propriedades no script do Player, altere-o para o seguinte e salve:











// Player.js
    //...
    properties: {
        // Main character's jump height
        jumpHeight: 0,
        // Main character's jump duration
        jumpDuration: 0,
        // Maximal movement speed
        maxMoveSpeed: 0,
        // Acceleration
        accel: 0,
    },
    //...

O Cocos Creator especifica que um nó possui propriedades que são escritas no bloco de código de propriedades, o que especificará como o personagem principal se move, e não precisamos nos preocupar com quais valores dessas propriedades estão no código, porque definiremos -los diretamente no painel Propriedades. Posteriormente, durante o desenvolvimento do jogo, podemos colocar todas as propriedades que precisam ser ajustadas a qualquer momento no bloco de propriedades.

Agora podemos adicionar o componente Player ao nó do personagem principal. Escolha o nó Player na árvore de nós, clique no botão Adicionar componente na parte inferior do painel Propriedades e escolha Componente personalizado -> Player para adicionar o componente Player ao nó do personagem principal.


Agora podemos ver o componente Player recém-adicionado no painel Propriedades do nó Player (o nó Player precisa ser selecionado na árvore de nós primeiro). Configure as propriedades relacionadas ao salto e movimento do personagem principal conforme imagem abaixo:


Apenas o valor da propriedade jumpDuration está em segundos e as outras propriedades estão em pixels. De acordo com a configuração atual do componente Player, nosso personagem principal terá uma altura de salto de 200 pixels, o tempo necessário para saltar para o ponto mais alto é de 0,3 segundos, sua velocidade máxima de movimento horizontal é de 400 pixels por segundo, sua aceleração horizontal é 350 pixels por segundo.

Todos esses valores numéricos são sugestões. Mais tarde, quando o jogo estiver em execução, você pode modificar esses valores numéricos no painel Propriedades a qualquer momento de acordo com sua preferência, sem necessidade de alterar nenhum código.



Writing code for jumping and movement
Next we add a method called runJumpAction under the properties: {...}, code block to make the main character jump.
// Player.js
    properties: {
        //...
    },

    runJumpAction () {
        // Jump up
        var jumpUp = cc.tween().by(this.jumpDuration, {y: this.jumpHeight}, {easing: 'sineOut'});
        // Jump down
        var jumpDown = cc.tween().by(this.jumpDuration, {y: -this.jumpHeight}, {easing: 'sineIn'});

        // Create a easing and perform actions in the order of "jumpUp", "jumpDown"
        var tween = cc.tween().sequence(jumpUp, jumpDown);
        // Repeat
        return cc.tween().repeatForever(tween);
    },

Aqui você precisa saber sobre o sistema cc.tween. No Cocos Creator, cc.tween fornece um método criado em cadeia que pode manipular qualquer objeto e facilitar qualquer uma das propriedades do objeto.

Por exemplo, no código acima, o método by () é usado para calcular o valor relativo da propriedade, que é o valor em mudança. Consulte a documentação do cc.tween e a classe Tween da documentação da API para obter mais detalhes.

Em seguida, chame o método runJumpAction que você acabou de adicionar ao método onLoad e chame start para iniciar a ação:

// Player.js
    onLoad: function () {
        // Initialize the jump action
        var jumpAction = this.runJumpAction();
        cc.tween(this.node).then(jumpAction).start()
    },
O método onLoad será executado imediatamente após o carregamento da cena, portanto as operações e a lógica relacionadas à inicialização devem ser colocadas no método onLoad. Primeiro passamos a ação de salto de loop para a variável jumpAction, depois a inserimos na fila onde cc.tween atenua o nó (personagem principal) e, finalmente, chamamos start para iniciar a execução de cc.tween, fazendo com que o personagem principal continue pulando .

Assim que as modificações do script forem feitas, lembre-se de salvá-lo e, em seguida, volte ao editor Criador e estaremos prontos para executar o jogo pela primeira vez!
Clique no botão Visualizar na parte superior do editor, que se parece com um botão "reproduzir":



O Criador abrirá automaticamente seu navegador padrão e executará o jogo nele. Agora devemos ver o personagem principal --- um monstro roxo pulando animada e continuamente na cena.


Manipulação de movimento

Um personagem principal que só pode pular tolamente para cima e para baixo no mesmo lugar não é muito promissor. Vamos adicionar entrada de teclado para o personagem principal, usando A e D para manipular sua direção de salto. Adicione a função de resposta de evento do teclado onKeyUp e onKeyDown abaixo do método runJumpAction:
 


// Player.js
    runJumpAction: function () {
        //...
    },

    onKeyDown (event) {
        // Set a flag when key pressed
        switch(event.keyCode) {
            case cc.macro.KEY.a:
                this.accLeft = true;
                break;
            case cc.macro.KEY.d:
                this.accRight = true;
                break;
        }
    },

    onKeyUp (event) {
        // Unset a flag when key released
        switch(event.keyCode) {
            case cc.macro.KEY.a:
                this.accLeft = false;
                break;
            case cc.macro.KEY.d:
                this.accRight = false;
                break;
        }
    },
Em seguida, modifique o método onLoad, ao qual adicionamos a chave de aceleração para a esquerda / direita e a velocidade horizontal atual do personagem principal. Por fim, chame cc.systemEvent e comece a ouvir a entrada do teclado depois que a cena for carregada:
 


// Player.js
    onLoad: function () {
        // Initialize jump action
        var jumpAction = this.runJumpAction();
        cc.tween(this.node).then(jumpAction).start()

        // Acceleration direction switch
        this.accLeft = false;
        this.accRight = false;
        // The main character's current horizontal velocity
        this.xSpeed = 0;

        // Initialize the keyboard input listening
        cc.systemEvent.on(cc.SystemEvent.EventType.KEY_DOWN, this.onKeyDown, this);
        cc.systemEvent.on(cc.SystemEvent.EventType.KEY_UP, this.onKeyUp, this);
    },

    onDestroy () {
        // Cancel keyboard input monitoring
        cc.systemEvent.off(cc.SystemEvent.EventType.KEY_DOWN, this.onKeyDown, this);
        cc.systemEvent.off(cc.SystemEvent.EventType.KEY_UP, this.onKeyUp, this);
    },
Aqui registramos as funções de resposta do teclado (onKeyDown, onKeyUp) no systemEvent, no qual usamos switch para determinar se a tecla A ou D do teclado está pressionada ou não, e se for pressionada, realizamos a operação correspondente.
Com a experiência de desenvolvimento do Android, é melhor entender que o ouvinte aqui é essencialmente o mesmo que o OnClickListener no Android, o Criador escuta os eventos globais do sistema via systemEvent. Para obter detalhes sobre como ouvir e despachar eventos de mouse, toque e personalizados, consulte os eventos Ouvir e iniciar.

Finalmente, modifique o método de atualização para adicionar configurações de aceleração, velocidade e a posição atual do personagem principal:

// Player.js
    update: function (dt) {
        // Update speed of each frame according to the current acceleration direction
        if (this.accLeft) {
            this.xSpeed -= this.accel * dt;
        } else if (this.accRight) {
            this.xSpeed += this.accel * dt;
        }
        // Restrict the movement speed of the main character to the maximum movement speed
        if ( Math.abs(this.xSpeed) > this.maxMoveSpeed ) {
            // If speed reach limit, use max speed with current direction
            this.xSpeed = this.maxMoveSpeed * this.xSpeed / Math.abs(this.xSpeed);
        }

        // Update the position of the main character according to the current speed
        this.node.x += this.xSpeed * dt;
    },
update será invocado uma vez por quadro depois que a cena for carregada, e geralmente colocamos o conteúdo lógico em update que precisa ser calculado frequentemente ou atualizado a tempo. Em nosso jogo, depois de obter a direção da aceleração da entrada do teclado, precisamos calcular a velocidade e a posição do personagem principal na atualização a cada quadro.

Depois de salvar o script, volte ao editor e clique no botão Visualizar na parte superior para verificar o resultado mais recente. Após abrir a visualização em seu navegador, clique na cena do jogo com o mouse (devido às restrições dos navegadores, a entrada do teclado só pode ser aceita após clicar na cena do jogo), então você pode pressionar os botões A e D para manipular o personagem principal para mover para a esquerda / direita!

O movimento está um pouco lento demais? O personagem principal não pula alto o suficiente? Espero estender a duração do salto? Sem problemas! Tudo isso pode ser ajustado a qualquer momento. Basta configurar valores de propriedade diferentes para o componente Player, então você pode ajustar o jogo como quiser. Aqui está um conjunto de configurações para referência:
Jump Height: 150
Jump Duration: 0.3
Max Move Speed: 400
Accel: 1000


Este conjunto de propriedades fará com que o personagem principal fique mais flexível, quanto a como escolher, depende do estilo de jogo que se deseja fazer.




Making stars

O personagem principal pode pular livremente agora, então precisamos definir uma meta para os jogadores. As estrelas aparecerão continuamente na cena e os jogadores precisam manipular o monstro para tocar as estrelas para coletar pontos. A estrela tocada pelo personagem principal desaparecerá e uma nova será imediatamente recriada em uma posição aleatória.

Criar Prefab
 
 
Quanto aos nós que precisam ser criados repetidamente, podemos salvá-los como um recurso Prefab, que pode ser um modelo para a geração dinâmica de nós. Para obter mais informações sobre Prefab, leia Prefab.
 
Em primeiro lugar, arraste e solte os ativos / texturas / textura de estrela do painel Ativos para o painel Cena em qualquer lugar que desejar. Precisamos apenas que a cena seja nossa bancada de trabalho para criar o Star Prefab. Após a criação, iremos deletar este nó da cena.
 
Não precisamos modificar a posição ou renderizar as propriedades da Estrela, mas para fazer a Estrela desaparecer quando tocada pelo personagem principal, precisamos adicionar um componente de script especial para a Estrela também. Pelo mesmo método de adição do script Player, adicione um script JavaScript denominado Star aos ativos / scripts.
 

 
Em seguida, clique duas vezes neste script para iniciar a edição. Apenas uma propriedade é necessária para o componente estrela estipular a distância para pontos de coleta pelo personagem principal. Modifique as propriedades e adicione o seguinte conteúdo:
 
 
 
// Star.js
    properties: {
        // When the distance between the star and main character is less than this value, collection of the point will be completed
        pickRadius: 0,
    },
 
Depois de salvar o script, adicione-o ao nó estrela recém-criado. Selecione o nó estrela na Árvore de nós e clique no botão Adicionar componente em Propriedades e selecione Componente personalizado -> Estrela, que é adicionado ao nó estrela recém-criado. Em seguida, defina o valor da propriedade Pick Radius para 60 e salve a cena.
 

 
As configurações necessárias para o Star Prefab estão concluídas. Agora arraste o nó estrela da Árvore de nós e solte-o na pasta de ativos em Ativos. Isso deve gerar um recurso Prefab denominado estrela, conforme mostrado abaixo.
 
Neste ponto, as configurações necessárias para o Prefab Star estão concluídas, e o recurso Prefab denominado star será gerado arrastando o nó estrela da Árvore de Nó para a pasta de ativos em Ativos, conforme mostrado abaixo:
 

 
 
Agora o nó da estrela pode ser excluído da cena. Podemos clicar duas vezes no recurso Prefab da estrela diretamente para editar.
 
Em seguida, podemos usar dinamicamente o recurso Prefab de estrelas no script para gerar estrelas
 
Adicionando script de controle de jogo
 
A geração de estrelas faz parte da lógica principal do jogo, portanto, precisamos adicionar um script JavaScript chamado Game como um script de lógica mestre do jogo. O script adicionará lógica para pontuação, falha do jogo e reinicialização.
 
Adicione um novo script de jogo dentro da pasta de recursos / scripts e clique duas vezes para abrir o script. Primeiro, adicione as propriedades necessárias para gerar estrelas:
 
 
 
 
 
 
 
 
 
 
// Game.js
    properties: {
        // This property quotes the PreFab resource of stars
        starPrefab: {
            default: null,
            type: cc.Prefab
        },
 
        // The random scale of disappearing time for stars
        maxStarDuration: 0,
        minStarDuration: 0,
 
        // Ground node for confirming the height of the generated star's position
        ground: {
            default: null,
            type: cc.Node
        },
 
        // Player node for obtaining the jump height of the main character and controlling the movement switch of the main character
        player: {
            default: null,
            type: cc.Node
        }
    },
 
 
Os iniciantes aqui podem se perguntar por que uma propriedade como starPrefab é incluída entre {} com a nova "propriedade" entre parênteses? Na verdade, esta é uma declaração completa de uma propriedade, anteriormente nossas declarações de propriedade eram incompletas e, em alguns casos, precisamos adicionar parâmetros à declaração de atributo que controlam como a propriedade é exibida no painel Propriedades e o comportamento da propriedade durante o processo de serialização da cena. Por exemplo:
 

 
properties: {
    score: {
        default: 0,
        displayName: "Score (player)",
        tooltip: "The score of player",
    }
}
O código acima define três parâmetros para o atributo score.
 
O código acima define três parâmetros para a propriedade score:
 
padrão - especifica que o valor padrão da pontuação é 0.
displayName - especifica que o nome da propriedade será exibido como Pontuação (player) no painel Propriedades.
Dica de ferramenta - Quando o mouse passa sobre a propriedade no painel Propriedades, exibe a dica de ferramenta correspondente.
 
Aqui estão os parâmetros comuns:
 
padrão: define o valor padrão da propriedade, que só é usado quando o componente é adicionado ao nó pela primeira vez
type: Para qualificar o tipo de dados de uma propriedade, consulte Referência avançada do CCClass: atributo type para obter detalhes.
visível: Defina como falso para não exibir a propriedade no painel Propriedades
serializable: Defina como false para não serializar (salvar) a propriedade
displayName: Exibe o nome da propriedade especificada no painel Propriedades
dica de ferramenta: adicione uma dica de ferramenta para a propriedade no painel Propriedades
 
Portanto, o código acima é fácil de entender:
 
starPrefab: {
    default: null,
    type: cc.Prefab
},
 
Primeiro, a propriedade starPrefab é declarada no componente Game, o valor padrão é nulo e o tipo que pode ser passado deve ser um tipo de recurso Prefab. Depois disso, as propriedades do terreno e do jogador também podem ser entendidas.
 
Depois de salvar o script, adicione o componente Game ao nó Canvas na árvore de nós (depois de escolher o nó Canvas, arraste o script para Propriedades ou clique no botão Adicionar componente em Propriedades e escolha Jogo no componente personalizado).
 

 
em seguida, arraste o recurso Star Prefab dos Ativos para a propriedade Star Prefab do componente Game recém-criado. Esta é a primeira vez que definimos uma referência a uma propriedade, e você pode arrastar um recurso ou um nó para essa propriedade (como o tipo cc.Prefab escrito aqui) apenas se for especificado como um tipo de referência quando a propriedade é declarado.
 

 
Em seguida, arraste e solte o solo e os nós Player da árvore de nós nas propriedades nomeadas de forma correspondente no componente Game do nó Canvas para completar a referência do nó.
 
Finalmente, defina os valores das propriedades Min Star Duration e Max Star Duration como 3 e 5. Mais tarde, ao gerar estrelas, escolheremos um valor aleatório entre esses dois valores, para a duração de cada estrela.
 
Gerar estrelas em uma posição aleatória
 

 
 
Em seguida, continuamos a modificar o script do jogo e adicionamos a lógica para gerar as estrelas após o método onLoad:
 
// Game.js
    onLoad: function () {
        // Obtain the anchor point of ground level on the y axis
        this.groundY = this.ground.y + this.ground.height/2; // "this.ground.top" may also work
        // Generate a new star
        this.spawnNewStar();
    },
 
    spawnNewStar: function() {
        // Generate a new node in the scene with a preset template
        var newStar = cc.instantiate(this.starPrefab);
        // Put the newly added node under the Canvas node
        this.node.addChild(newStar);
        // Set up a random position for the star
        newStar.setPosition(this.getNewStarPosition());
    },
 
    getNewStarPosition: function () {
        var randX = 0;
        // According to the position of the ground level and the main character's jump height, randomly obtain an anchor point of the star on the y axis
        var randY = this.groundY + Math.random() * this.player.getComponent('Player').jumpHeight + 50;
        // According to the width of the screen, randomly obtain an anchor point of star on the x axis
        var maxX = this.node.width/2;
        randX = (Math.random() - 0.5) * 2 * maxX;
        // Return to the anchor point of the star
        return cc.v2(randX, randY);
    },
 
Aqui estão algumas coisas a serem observadas.
 
A propriedade y sob o nó corresponde à coordenada y do ponto de ancoragem, porque o ponto de ancoragem é padronizado para o centro do nó, então a coordenada y do solo precisa adicionar metade da altura do solo.
instantiate () é usado para clonar o objeto especificado de qualquer tipo ou para instanciar um novo nó do Prefab. O valor de retorno é Node ou Object.
O efeito do método addChild sob o nó é construir um novo nó no próximo nível do nó, de forma que o novo nó seja exibido acima do nó
O método setPosition () sob o nó é definir a posição do nó no sistema de coordenadas do nó pai, e você pode definir os pontos de coordenadas de duas maneiras:
A primeira é passar dois valores x e y.
O segundo é Passar em cc.v2 (x, y) ou cc.v3 (x, y, z) (um objeto do tipo cc.Vec2 ou cc.Vec3)
O efeito do método getComponent sob o nó é obter as referências do componente que são montadas no nó
Após salvar o script, volte ao editor e clique no botão Visualizar. Então, no navegador, você verá que uma estrela é gerada dinamicamente depois que o jogo é iniciado! Pelo mesmo método, você pode gerar dinamicamente qualquer nó predefinido com um modelo Prefab no jogo.
 
 
 
 

 
 
Adicionando a ação de tocar e coletar estrelas do personagem principal
 
Agora vamos adicionar a lógica do comportamento do personagem principal de coletar estrelas. O essencial aqui é que as estrelas precisam obter a posição do nó do personagem principal a qualquer momento para determinar se a distância entre eles é menor do que a distância coletável. Como obtemos a referência do nó do personagem principal? Não se esqueça de que fizemos duas coisas antes:
 
Existe uma propriedade chamada player no componente Game, que salvou a referência do nó do personagem principal.
Cada estrela é gerada dinamicamente no script do jogo.
Portanto, precisamos apenas passar a instância para o componente de script Star e salvá-la quando o script do jogo gerar a instância do nó Star, e então podemos acessar o nó do personagem principal por meio de game.player a qualquer momento.
 
Vamos abrir o script do jogo e adicionar newStar.getComponent ('Star'). Game = this; no final do método spawnNewStar, da seguinte forma:
// Game.js
    spawnNewStar: function() {
        // ...
 
        // Save a reference of the Game object on the Star script component
        newStar.getComponent('Star').game = this;
    },
 
 
Abra o script Star depois de salvar, e agora podemos usar o nó Player referenciado no componente Game para determinar a distância. Adicione métodos chamados getPlayerDistance e onPicked após o método onLoad.
 
// Star.js
    getPlayerDistance: function () {
        // Determine the distance according to the position of the Player node
        var playerPos = this.game.player.getPosition();
 
        // Calculate the distance between two nodes according to their positions
        var dist = this.node.position.sub(playerPos).mag();
        return dist;
    },
 
    onPicked: function() {
        // When the stars are being collected, invoke the interface in the Game script to generate a new star
        this.game.spawnNewStar();
 
        // Then destroy the current star's node
        this.node.destroy();
    },
O método getPosition () em Node retorna a posição (x, y) do nó no sistema de coordenadas do nó pai, que é um objeto do tipo Vec2. E então chamar o método destroy () no Node pode destruir o nó.
 
Em seguida, adicione a distância do juiz por quadro no método de atualização. Se a distância for menor que a distância de coleta especificada pela propriedade pickRadius, o comportamento de coleta é executado:
 
// Star.js
    update: function (dt) {
        // Determine if the distance between the Star and main character is less than the collecting distance for each frame
        if (this.getPlayerDistance() < this.pickRadius) {
            // Invoke collecting behavior
            this.onPicked();
            return;
        }
    },
 
Salve o script, pressionando as teclas A e D para controlar o personagem principal para se mover, você pode ver que quando o personagem principal se aproxima da estrela, a estrela irá desaparecer e uma nova será gerada em uma posição aleatória!
 
Adicionando pontuação
 

 
O pequeno monstro trabalha tanto para coletar as estrelas, como ele pode não ser recompensado? Agora, vamos adicionar a lógica e a exibição de recompensas de pontuação ao coletar Estrelas.
 

 
 
 
 
 
 
 
 
Adicionando um rótulo de pontuação (rótulo)
 

 
A pontuação começará em 0 quando o jogo começar, e 1 ponto será adicionado para cada Estrela que o Monstro coletar. Para exibir a pontuação, devemos primeiro criar um nó Label. Escolha o nó Canvas na árvore de nós, clique com o botão direito e escolha Create -> Create Renderer Nodes -> Node With Label  opção Nó com rótulo no menu, um novo nó Rótulo será criado sob o nó Canvas, com a ordem da hierarquia na parte inferior. Em seguida, configuramos esse nó Label seguindo as etapas abaixo:
 

Altere o nome do nó para pontuação.
Selecione o nó de pontuação na árvore de nós e defina o X, Y da propriedade de posição como (0, 180).
Edite a propriedade String do componente Label e insira Score: 0.
Observação: a propriedade String do componente Label não reconhecerá dois pontos chineses quando uma fonte bitmap for adicionada.
 
Defina a propriedade Font Size do componente Label como 50.
Arraste o recurso de fonte de bitmap assets / mikado_outline_shadow de Assets (observe que o ícone é bmfont) para a propriedade Font do componente Label, substitua a fonte do texto pela fonte de bitmap em nosso recurso de projeto
O efeito da configuração concluída é mostrado na figura a seguir:
 

 
 
Adicionando lógica de pontuação ao script do jogo
 
Colocaremos a lógica de pontuação e atualização da exibição da pontuação no script do jogo.
Abra o script do jogo para começar a editar. Primeiro, adicione uma propriedade de referência do rótulo de exibição de pontuação ao final do bloco de propriedades:
 
 
 
// Game.js
    properties: {
        // ...
 
        // Reference of score label
        scoreDisplay: {
            default: null,
            type: cc.Label
        }
    },
 
Em seguida, adicione a inicialização da variável usada para pontuação ao final do método onLoad:
 

 
// Game.js
    onLoad: function () {
        // ...
        // Initialize scoring
        this.score = 0;
    },
 
 
 
 
 
Em seguida, adicione um novo método chamado gainScore após o método de atualização:
 

// Game.js
    gainScore: function () {
        this.score += 1;
        // Update the words of the scoreDisplay Label
        this.scoreDisplay.string = 'Score: ' + this.score;
    },
 
Depois de salvar o script do jogo, volte ao editor, selecione o nó Canvas na árvore de nós e arraste o nó da pontuação que adiciona antes à propriedade Exibição da pontuação do componente Jogo nas propriedades.
 

 
Invoque a lógica de pontuação
 

Em seguida, precisamos chamar a lógica de pontuação do script Game no script Star. Abra o script Star e adicione a invocação de gainScore ao método onPicked.
 

 
// Star.js
    onPicked: function() {
        // When the stars are being collected, invoke the interface in the Game script to generate a new star
        this.game.spawnNewStar();
 
        // Invoke the scoring method of the Game script
        this.game.gainScore();
 
        // Then destroy the current star's node
        this.node.destroy();
    },
Depois de salvar o script, você deve voltar ao editor para que as alterações do script tenham efeito. Em seguida, clique em Visualizar e você verá que a pontuação exibida logo acima da tela agora aumenta ao coletar estrelas!
 

After saving the script you must go back to the editor for the script changes to take effect. Then click on the Preview and you can see that the score displayed right above the screen now increases when collecting stars!
 

 
Failure judgment and restart
 
Agora nosso jogo começou a tomar forma. Mas não importa quantas pontuações alguém consiga, um jogo sem a possibilidade de falha não proporcionará aos jogadores nenhuma satisfação. Agora vamos adicionar a ação de desaparecimento regular da estrela. E se todas as estrelas desaparecerem, o jogo será visto como um fracasso. Em outras palavras, os jogadores precisam terminar de coletar a estrela antes que ela desapareça e repetir esse procedimento incessantemente para terminar o loop do método de jogo.
 

 
Adicionando a lógica de desaparecer em um tempo limitado para a estrela
 

Abra o script do jogo e adicione a declaração de variável necessária para contar o tempo antes de invocar spawnNewStar do método onLoad:
 

 
// Game.js
    onLoad: function () {
        // ...
 
        // Initialize timer
        this.timer = 0;
        this.starDuration = 0;
 
        // Generate a new star
        this.spawnNewStar();
 
        // Initialize scoring
        this.score = 0;
    },
Em seguida, adicione a lógica de redefinir o cronômetro para o final do método spawnNewStar, em que this.minStarDuration e this.maxStarDuration são propriedades do componente Game que foi declarado no início, para especificar um intervalo aleatório de tempo para as estrelas desaparecerem .
// Game.js
    spawnNewStar: function() {
        // ...
 
        // Reset timer, randomly choose a value according the scale of star duration
        this.starDuration = this.minStarDuration + Math.random() * (this.maxStarDuration - this.minStarDuration);
        this.timer = 0;
    },
Adicione lógica ao método de atualização para atualizar o cronômetro e determinar se ele expirou.
 

// Game.js
    update: function (dt) {
        // Update timer for each frame, when a new star is not generated after exceeding duration
 
        // Invoke the logic of game failure
        if (this.timer > this.starDuration) {
            this.gameOver();
            return;
        }
        this.timer += dt;
    },
Finalmente, adicione um método gainScore após o método gameOver para recarregar a cena se o jogo falhar.
 

// Game.js
    gameOver: function () {
        // Stop the jumping action of the Player node
        this.player.stopAllActions(); 
        // reload the "game" scene
        cc.director.loadScene('game');
    }
O iniciante precisa saber que cc.director é um único objeto que gerencia o processo lógico do seu jogo. Como cc.director é um único instante, você não precisa chamar nenhum construtor ou criar funções, a maneira padrão de usá-lo é chamando cc.director.methodName (). Por exemplo, o cc.director.loadScene ('game') aqui é para recarregar a cena do jogo, o que significa que o jogo começa novamente. E o método stopAllActions no nó invalidará toda a ação no nó.
 
 
 
 
// Star.js
    update: function() {
        // ...
 
        // Update the transparency of the star according to the timer in the Game script
        var opacityRatio = 1 - this.game.timer/this.game.starDuration;
        var minOpacity = 50;
        this.node.opacity = minOpacity + Math.floor(opacityRatio * (255 - minOpacity));
    }
Salve o script Star e a lógica do método de jogo do nosso jogo estará pronta! Agora volte ao editor e clique no botão Visualizar, veremos um jogo qualificado com um método de jogo principal, mecanismo de incentivo e mecanismo de falha no navegador.
 

Adicionar efeitos sonoros
 

Embora muitas pessoas ignorem o som ao jogar jogos para celular, para completar o fluxo de trabalho apresentado neste tutorial, ainda temos que complementar a tarefa de adicionar efeitos sonoros.
 

 
Efeito de som de salto
 

Em primeiro lugar, adicione o efeito de som de salto. Abra o script do Player e adicione uma propriedade jumpAudio às propriedades para fazer referência ao recurso do arquivo de som:
 

// Player.js
    properties: {
        // ...
 
        // Jumping sound effect resource
        jumpAudio: {
            default: null,
            type: cc.AudioClip
        },
    },
Em seguida, reescreva o método runJumpAction, insira o retorno de chamada para reproduzir o efeito de som e reproduza o som adicionando um método playJumpSound:
 

// Player.js
    runJumpAction: function () {
        // Jump up
        var jumpUp = cc.tween().by(this.jumpDuration, {y: this.jumpHeight}, {easing: 'sineOut'});
 
        // Jump down
        var jumpDown = cc.tween().by(this.jumpDuration, {y: -this.jumpHeight}, {easing: 'sineIn'});
 
        // Create a easing
        var tween = cc.tween()
                        // perform actions in the order of "jumpUp", "jumpDown"
                        .sequence(jumpUp, jumpDown)
                        // Add a callback function to invoke the "playJumpSound()" method we define after the action is finished
                        .call(this.playJumpSound, this);
 
        // Repeat unceasingly
        return cc.tween().repeatForever(tween);
    },
 
    playJumpSound: function () {
        // Invoke sound engine to play the sound
        cc.audioEngine.playEffect(this.jumpAudio, false);
    },
Efeito de som de pontuação
 

Depois de salvar o script do Player, abra o script do jogo para adicionar o efeito de som de pontuação. Em primeiro lugar, ainda adicionamos uma propriedade scoreAudio nas propriedades para fazer referência ao recurso do arquivo de som:
 

 
 
 
 
 
 
 
 
 
 
// Game.js
    properties: {
        // ...
 
        // Scoring sound effect resource
        scoreAudio: {
            default: null,
            type: cc.AudioClip
        }
    },
 
Em seguida, adicione o código de reprodução do som ao final do método gainScore:
 

// Game.js
    gainScore: function () {
        this.score += 1;
        // Update the words of the scoreDisplay Label
        this.scoreDisplay.string = 'Score: ' + this.score.toString();
        // Play the scoring sound effect
        cc.audioEngine.playEffect(this.scoreAudio, false);
    },
 
 
Salve o script do jogo e volte para a árvore de nós do editor. Escolha o nó Player e arraste o recurso assets / audio / jump de Assets para a propriedade Jump Audio do componente Player.
Em seguida, escolha o nó Canvas, arraste o recurso assets / audio / score para a propriedade Score Audio do componente Game.
 
Então terminamos! A hierarquia do nó e as propriedades de cada componente-chave no estado concluído são as seguintes:



 
Agora podemos desfrutar totalmente do jogo recém-terminado, quantas pontuações você consegue? Não se esqueça de que você pode alterar os parâmetros do jogo, como controle de movimento e tempo de permanência das estrelas nos componentes Player e Game a qualquer momento, para ajustar rapidamente a dificuldade do jogo. Você precisa salvar a cena após modificar as propriedades do componente para que os valores modificados sejam registrados.
 
