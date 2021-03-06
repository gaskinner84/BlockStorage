---

copyright:
  years: 2014, 2019
lastupdated: "2019-02-05"

keywords:

subcollection: BlockStorage

---
{:new_window: target="_blank"}
{:tip: .tip}
{:note: .note}
{:important: .important}
{:codeblock: .codeblock}

# Conectando-se a LUNs iSCSI no Microsoft Windows
{: #mountingWindows}

Antes de iniciar, certifique-se de que o host que está acessando o volume {{site.data.keyword.blockstoragefull}} tenha sido autorizado por meio do [{{site.data.keyword.slportal}} ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com/){:new_window}.

1. Na página de listagem do {{site.data.keyword.blockstorageshort}}, localize o novo volume e clique em **Ações**. Clique em **Autorizar host**.
2. Na lista, selecione o host ou os hosts que devem acessar o volume e clique em **Enviar**.

Como alternativa, é possível autorizar o host por meio do SLCLI.
```
# slcli block access-authorize --help Usage: slcli block access-authorize [OPTIONS] VOLUME_ID

Options:
  -h, --hardware-id TEXT    The id of one SoftLayer_Hardware to authorize
  -v, --virtual-id TEXT     The id of one SoftLayer_Virtual_Guest to authorize
  -i, --ip-address-id TEXT  The id of one SoftLayer_Network_Subnet_IpAddress
                            to authorize
  --ip-address TEXT         An IP address to authorize
  --help                    Show this message and exit.
```
{:codeblock}

## Montando  {{site.data.keyword.blockstorageshort}}  Volumes
{: #mountWin}

A seguir estão as etapas necessárias para conectar uma instância de Cálculo do {{site.data.keyword.BluSoftlayer_full}} baseada no Windows a um número de unidade lógica (LUN) de Small Computer System Interface (iSCSI) da internet de Multipath input/output (MPIO). O exemplo é baseado no Windows Server 2012. As etapas podem ser ajustadas para outras versões do Windows de acordo com a documentação do fornecedor do sistema operacional (S.O.).

### Configurando o recurso MPIO

1. Inicie o Gerenciador do servidor e navegue para **Gerenciar**, **Incluir funções e recursos**.
2. Clique em **Avançar** para o menu Recursos.
3. Role para baixo e marque **Multipath I/O**.
4. Clique em **Instalar** para instalar o MPIO no servidor host.
![Incluindo funções e recursos no Server Manager](/images/Roles_Features.png)

### Incluindo suporte iSCSI para MPIO

1. Abra a janela Propriedades de MPIO clicando em **Iniciar**, apontando para **Ferramentas administrativas** e clicando em **MPIO**.
2. Clique em **Descobrir caminhos múltiplos**.
3. Marque **Incluir suporte para dispositivos iSCSI** e, em seguida, clique em **Incluir**. Quando solicitado a reiniciar o computador, clique em
**Sim**.

No Windows Server 2008, a inclusão de suporte para o iSCSI permite ao Microsoft Device Specific Module (MSDSM)
reivindicar todos os dispositivos iSCSI para MPIO, o que requer uma conexão com um destino iSCSI primeiro.
{:note}

### Configurando o Inicializador iSCSI

1. Inicie o Inicializador iSCSI por meio do Gerenciador do servidor e selecione **Ferramentas**, **Inicializador iSCSI**.
2. Clique na guia **Configuração**.
    - O campo Nome do inicializador poderá já estar preenchido com uma entrada semelhante a `iqn.1991-05.com.microsoft:`.
    - Clique em **Mudar** para substituir os valores existentes pelo nome qualificado de
iSCSI (IQN). ![Propriedades do inicializador iSCSI](/images/iSCSI.png)

      O nome do IQN pode ser obtido por meio da tela Detalhes do {{site.data.keyword.blockstorageshort}} no [{{site.data.keyword.slportal}} ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com/){:new_window}.
      {: tip}

    - Clique na guia **Descoberta** e clique em **Descobrir portal**.
    - Insira o endereço IP de seu destino iSCSI e deixe a Porta no valor padrão de 3260.
    - Clique em **Avançado** para abrir a janela Configurações avançadas.
    - Selecione **Ativar logon do CHAP** para ativar a autenticação do CHAP.
![Ativar login do CHAP](/images/Advanced_0.png)

    Os campos Nome e Segredo de destino fazem distinção entre maiúsculas e minúsculas.
    {:important}
         - No campo **Nome**, exclua quaisquer entradas existentes e insira o nome do usuário por meio do [ {{site.data.keyword.slportal}} ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com/){:new_window}.
         - No campo **Segredo do destino**, insira a senha por meio do [{{site.data.keyword.slportal}} ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com/){:new_window}.
    - Clique em **OK** nas janelas **Configurações avançadas** e **Descobrir portal de destino** para voltar à tela principal Propriedades do inicializador iSCSI. Se você receber erros de autenticação, verifique as entradas de nome de usuário e senha.
    ![Destino inativo](/images/Inactive_0.png)

    O nome do destino aparece na seção Destinos descobertos com um status `Inactive`.
    {:note}


### Ativando o destino

1. Clique em **Conectar** para conectar-se ao destino.
2. Marque a caixa de seleção **Ativar caminhos múltiplos** para ativar a E/S de caminhos múltiplos para o destino.
<br/>
   ![Ativar caminhos múltiplos](/images/Connect_0.png)
3. Clique em **Avançado** e selecione **Ativar logon do CHAP**.
</br>
   ![Ativar o CHAP](/images/chap_0.png)
4. Insira o nome do usuário no campo Nome e insira a senha no campo Segredo de destino.

   Os valores de campo Nome e Segredo de destino podem ser obtidos por meio da tela Detalhes do {{site.data.keyword.blockstorageshort}}.
   {:tip}
5. Clique em **OK** até que a janela **Propriedades do inicializador iSCSI** seja exibida. O status do destino na seção **Destinos descobertos** muda de **Inativo** para **Conectado**.
![Status Conectado](/images/Connected.png)


### Configurando o MPIO no Inicializador iSCSI

1. Inicie o Inicializador iSCSI e, na guia Destinos, clique em **Propriedades**.
2. Clique em **Incluir sessão** na janela Propriedades para abrir a janela Conectar ao destino.
3. Na caixa de diálogo Conectar-se ao destino, selecione a caixa de opção **Ativar caminhos múltiplos** e clique em **Avançado**.
  ![Target](/images/Target.png)

4. Na janela Configurações avançadas ![Configurações](/images/Settings.png)
   - Na lista Adaptador local, selecione Inicializador iSCSI da Microsoft.
   - Na lista IP do inicializador, selecione o endereço IP do host.
   - Na lista IP do portal de destino, selecione o IP da interface do dispositivo.
   - Clique na caixa de seleção **Ativar logon do CHAP**
   - Insira os valores secretos de Nome e Destino obtidos no portal e clique em **OK**.
   - Clique em **OK** na janela Conectar-se ao destino para voltar para a janela
Propriedades.

5. Clique em **Propriedades**. Na caixa de diálogo Propriedades, clique em **Incluir
sessão** novamente para incluir o segundo caminho.
6. Na janela Conectar ao destino, selecione a caixa de opção **Ativar caminhos múltiplos**. Clique em **Avançado**.
7. Na janela Configurações avançadas,
   - Na lista Adaptador local, selecione Inicializador iSCSI da Microsoft.
   - Na lista IP do inicializador, selecione o endereço IP correspondente ao host. Nesse caso, você está conectando duas
interfaces de rede no dispositivo de armazenamento a uma única interface de rede no host. Portanto, essa interface é a mesma que foi fornecida para a primeira sessão.
   - Na lista IP do portal de destino, selecione o endereço IP para a segunda interface de dados que é permitida no dispositivo de armazenamento.

     É possível localizar o segundo endereço IP na tela Detalhes do {{site.data.keyword.blockstorageshort}}
no [{{site.data.keyword.slportal}}
![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com/){:new_window}.
      {: tip}
   - Clique na caixa de seleção **Ativar logon do CHAP**
   - Insira os valores secretos de Nome e Destino obtidos no portal e clique em **OK**.
   - Clique em **OK** na janela Conectar-se ao destino para voltar para a janela
Propriedades.
8. Agora a janela Propriedades exibe mais de uma sessão dentro da área de janela Identificador. Você tem mais de
uma sessão no armazenamento iSCSI.

   Se o host tiver múltiplas interfaces que você deseja conectar ao armazenamento ISCSI, será possível
configurar outra conexão com o endereço IP do outro NIC no campo IP do inicializador. No entanto, certifique-se de autorizar o segundo endereço IP do inicializador no [{{site.data.keyword.slportal}} ![Ícone de link externo](../../icons/launch-glyph.svg "Ícone de link externo")](https://control.softlayer.com/){:new_window} antes de tentar fazer a conexão.
   {:note}
9. Na janela Propriedades, clique em **Dispositivos** para abrir a janela Dispositivos. O nome da interface do dispositivo começa com `mpio`. <br/>
  ![Dispositivos](/images/Devices.png)

10. Clique em **MPIO** para abrir a janela **Detalhes do dispositivo**. É possível escolher as políticas de balanceamento de carga para o MPIO nessa janela e ela mostrará os caminhos para o iSCSI. Neste exemplo, dois caminhos são mostrados como disponíveis para o MPIO com uma política de balanceamento de carga de Round Robin com Subconjunto.
  ![A janela Detalhes do dispositivo mostra dois caminhos disponíveis para o MPIO com uma política de balanceamento de carga de Round Robin com Subconjunto](/images/DeviceDetails.png)

11. Clique em **OK** várias vezes para sair do Inicializador iSCSI.



## Verificando se o MPIO está configurado corretamente em sistemas operacionais Windows
{: #verifyMPIOWindows}

Para verificar se o MPIO do Windows está configurado, deve-se primeiro assegurar que o Complemento MPIO esteja ativado e reiniciar o servidor.

![Roles_Features_0](/images/Roles_Features_0.png)

Quando a reinicialização estiver concluída e o Dispositivo de armazenamento incluído, será possível verificar se o MPIO está configurado e funcionando. Para fazer isso, consulte **Detalhes do
dispositivo de destino** e clique em **MPIO**:
![DeviceDetails_0](/images/DeviceDetails_0.png)

Se o MPIO não foi configurado corretamente, o dispositivo de armazenamento poderá ser desconectado e aparecer desativado quando ocorrer uma indisponibilidade de rede ou quando as equipes do {{site.data.keyword.BluSoftlayer_full}} executarem a manutenção. O MPIO assegura um nível extra de conectividade durante esses eventos e mantém uma sessão estabelecida com operações de leitura/gravação ativas indo para o LUN.

## Desmontando volumes  {{site.data.keyword.blockstorageshort}}
{: #unmountingWin}

A seguir estão as etapas necessárias para desconectar uma instância de cálculo do {{site.data.keyword.Bluemix_short}} baseada no Windows de um LUN do iSCSI de MPIO. O exemplo é baseado no Windows Server 2012. As etapas podem ser ajustadas para outras versões do Windows de acordo com a documentação do fornecedor do S.O.

### Iniciando o Inicializador iSCSI

1. Clique na guia **Destinos**.
2. Selecione os destinos que você deseja remover e clique em **Desconectar**.

### Removendo destinos
Essa etapa é opcional para quando você não precisar mais acessar os destinos iSCSI.

1. Clique em **Descoberta** no Inicializador iSCSI.
2. Destaque o portal de destino que está associado a seu volume de armazenamento e clique em **Remover**.
