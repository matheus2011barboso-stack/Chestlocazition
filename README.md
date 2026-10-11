-- Serviços do Roblox
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer

-- Remove GUI antiga se já existir
if CoreGui:FindFirstChild("ChestFinderGUI") then
    CoreGui.ChestFinderGUI:Destroy()
end

-- ScreenGui Principal
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ChestFinderGUI"
ScreenGui.ResetOnSpawn = false

pcall(function()
    ScreenGui.Parent = CoreGui
end)
if not ScreenGui.Parent then
    ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

-- Janela Principal
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 320, 0, 280)
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -140)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.ClipsDescendants = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

-- Barra de Título (Mover GUI)
local TitleBar = Instance.new("Frame")
TitleBar.Name = "TitleBar"
TitleBar.Size = UDim2.new(1, 0, 0, 35)
TitleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
TitleBar.BorderSizePixel = 0
TitleBar.Parent = MainFrame

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 10)
TitleCorner.Parent = TitleBar

local TitleText = Instance.new("TextLabel")
TitleText.Size = UDim2.new(1, -40, 1, 0)
TitleText.Position = UDim2.new(0, 10, 0, 0)
TitleText.BackgroundTransparency = 1
TitleText.Text = "Rastreador de Baús (Limpo)"
TitleText.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleText.TextSize = 13
TitleText.Font = Enum.Font.SourceSansBold
TitleText.TextXAlignment = Enum.TextXAlignment.Left
TitleText.Parent = TitleBar

-- Botão Fechar [X]
local CloseBtn = Instance.new("TextButton")
CloseBtn.Size = UDim2.new(0, 25, 0, 25)
CloseBtn.Position = UDim2.new(1, -30, 0, 5)
CloseBtn.BackgroundColor3 = Color3.fromRGB(220, 50, 50)
CloseBtn.Text = "X"
CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CloseBtn.Font = Enum.Font.SourceSansBold
CloseBtn.TextSize = 14
CloseBtn.Parent = TitleBar

local CloseCorner = Instance.new("UICorner")
CloseCorner.CornerRadius = UDim.new(0, 6)
CloseCorner.Parent = CloseBtn

-- Scroll para a Lista de Baús
local ScrollContainer = Instance.new("ScrollingFrame")
ScrollContainer.Name = "ScrollContainer"
ScrollContainer.Size = UDim2.new(1, -20, 1, -85)
ScrollContainer.Position = UDim2.new(0, 10, 0, 40)
ScrollContainer.BackgroundTransparency = 1
ScrollContainer.BorderSizePixel = 0
ScrollContainer.ScrollBarThickness = 6
ScrollContainer.ScrollBarImageColor3 = Color3.fromRGB(80, 80, 100)
ScrollContainer.CanvasSize = UDim2.new(0, 0, 0, 0)
ScrollContainer.Parent = MainFrame

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = ScrollContainer
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
UIListLayout.Padding = UDim.new(0, 6)

UIListLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
    ScrollContainer.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y + 10)
end)

-- Botão de Atualizar Lista
local RefreshBtn = Instance.new("TextButton")
RefreshBtn.Size = UDim2.new(1, -20, 0, 30)
RefreshBtn.Position = UDim2.new(0, 10, 1, -35)
RefreshBtn.BackgroundColor3 = Color3.fromRGB(45, 85, 125)
RefreshBtn.Text = "🔄 Atualizar Lista"
RefreshBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
RefreshBtn.Font = Enum.Font.SourceSansBold
RefreshBtn.TextSize = 14
RefreshBtn.Parent = MainFrame

local RefreshCorner = Instance.new("UICorner")
RefreshCorner.CornerRadius = UDim.new(0, 6)
RefreshCorner.Parent = RefreshBtn

--------------------------------------------------------------------------------
-- SISTEMA DE MOVER A GUI (DRAGGABLE)
--------------------------------------------------------------------------------
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

TitleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

TitleBar.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

--------------------------------------------------------------------------------
-- TRADUÇÃO E LIMPEZA DE NOMES PARA PORTUGUÊS
--------------------------------------------------------------------------------
local function TraduzirParaPtBr(nomeOriginal)
    local nome = nomeOriginal
    local lower = string.lower(nome)

    -- Limpa caminhos técnicos
    if string.find(nome, "/") then
        local partes = {}
        for parte in string.gmatch(nome, "[^/]+") do
            table.insert(partes, parte)
        end
        nome = partes[#partes]
        lower = string.lower(nome)
    end

    if string.find(lower, "dark") or string.find(lower, "shadow") or string.find(lower, "escuridao") then
        return "Baú Escuro"
    elseif string.find(lower, "light") or string.find(lower, "bright") or string.find(lower, "luz") then
        return "Baú de Luz"
    elseif string.find(lower, "gold") or string.find(lower, "golden") or string.find(lower, "ouro") then
        return "Baú Dourado"
    elseif string.find(lower, "silver") or string.find(lower, "prata") then
        return "Baú Prateado"
    elseif string.find(lower, "rare") or string.find(lower, "raro") then
        return "Baú Raro"
    elseif string.find(lower, "epic") or string.find(lower, "epico") then
        return "Baú Épico"
    elseif string.find(lower, "legendary") or string.find(lower, "lendario") then
        return "Baú Lendário"
    elseif string.find(lower, "mythic") or string.find(lower, "mitico") then
        return "Baú Mítico"
    elseif string.find(lower, "normal") or string.find(lower, "common") or string.find(lower, "comum") or string.find(lower, "cube") then
        return "Baú Normal"
    else
        nome = string.gsub(string.lower(nome), "chest", "Baú")
        nome = string.gsub(string.lower(nome), "bau", "Baú")
        if not string.find(string.lower(nome), "baú") then
            nome = "Baú " .. nome
        end
        return nome
    end
end

--------------------------------------------------------------------------------
-- SISTEMA DE ESP E ROTAÇÃO POR TIPO DE BAÚ
--------------------------------------------------------------------------------
local ativoHighlight = nil
local ativoBillboard = nil
local tipoAtivo = nil
local indicesAtuais = {}
local botoesCriados = {}

local function LimparESPAtual()
    if ativoHighlight then
        ativoHighlight:Destroy()
        ativoHighlight = nil
    end
    if ativoBillboard then
        ativoBillboard:Destroy()
        ativoBillboard = nil
    end
    for _, b in pairs(botoesCriados) do
        if b and b.Parent then
            local nomeOriginal = b:GetAttribute("NomeTipo") or ""
            local qtd = b:GetAttribute("Quantidade") or 0
            b.Text = "📦 " .. nomeOriginal .. " (" .. qtd .. ")"
            b.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
            b.TextColor3 = Color3.fromRGB(230, 230, 230)
        end
    end
end

local function AtualizarListaBaus()
    LimparESPAtual()
    tipoAtivo = nil

    for _, child in pairs(ScrollContainer:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    botoesCriados = {}

    local gruposDeBaus = {}

    for _, v in pairs(Workspace:GetDescendants()) do
        local nomeLower = string.lower(v.Name)
        if string.find(nomeLower, "chest") or string.find(nomeLower, "bau") or string.find(nomeLower, "baú") or string.find(nomeLower, "cube") then
            if v:IsA("BasePart") or v:IsA("Model") then
                if not v:FindFirstAncestorOfClass("Player") then
                    -- Pega o modelo pai se existir para unificar peças do mesmo baú
                    local objetoAlvo = v
                    if v:IsA("BasePart") and v.Parent and v.Parent:IsA("Model") then
                        local nomePai = string.lower(v.Parent.Name)
                        if string.find(nomePai, "chest") or string.find(nomePai, "bau") or string.find(nomePai, "baú") or string.find(nomePai, "cube") then
                            objetoAlvo = v.Parent
                        end
                    end

                    local nomeTraduzido = TraduzirParaPtBr(objetoAlvo.Name)
                    if not gruposDeBaus[nomeTraduzido] then
                        gruposDeBaus[nomeTraduzido] = {}
                    end

                    -- Evita duplicar o mesmo baú na lista
                    local jaExiste = false
                    for _, cad in ipairs(gruposDeBaus[nomeTraduzido]) do
                        if cad == objetoAlvo then
                            jaExiste = true
                            break
                        end
                    end

                    if not jaExiste then
                        table.insert(gruposDeBaus[nomeTraduzido], objetoAlvo)
                    end
                end
            end
        end
    end

    local contador = 0

    for nomeTipo, listaDeObjetos in pairs(gruposDeBaus) do
        contador = contador + 1
        
        local validos = {}
        for _, obj in ipairs(listaDeObjetos) do
            if obj and obj.Parent then
                table.insert(validos, obj)
            end
        end
        gruposDeBaus[nomeTipo] = validos

        if not indicesAtuais[nomeTipo] then
            indicesAtuais[nomeTipo] = 1
        end

        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, -10, 0, 32)
        btn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
        btn.Text = "📦 " .. nomeTipo .. " (" .. #validos .. ")"
        btn.TextColor3 = Color3.fromRGB(230, 230, 230)
        btn.Font = Enum.Font.SourceSansBold
        btn.TextSize = 13
        btn.Parent = ScrollContainer

        btn:SetAttribute("NomeTipo", nomeTipo)
        btn:SetAttribute("Quantidade", #validos)

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 6)
        corner.Parent = btn

        botoesCriados[nomeTipo] = btn

        btn.MouseButton1Click:Connect(function()
            if tipoAtivo == nomeTipo then
                LimparESPAtual()
                tipoAtivo = nil
                return
            end

            LimparESPAtual()

            local atualizados = {}
            for _, obj in ipairs(listaDeObjetos) do
                if obj and obj.Parent then
                    table.insert(atualizados, obj)
                end
            end
            gruposDeBaus[nomeTipo] = atualizados

            if #atualizados == 0 then
                btn.Text = "❌ " .. nomeTipo .. " (0)"
                return
            end

            local idx = indicesAtuais[nomeTipo]
            if idx > #atualizados then
                idx = 1
                indicesAtuais[nomeTipo] = 1
            end

            tipoAtivo = nomeTipo
            local alvo = atualizados[idx]

            indicesAtuais[nomeTipo] = idx + 1

            if alvo then
                -- 1. Cria o ESP de brilho
                local hl = Instance.new("Highlight")
                hl.Name = "ChestESP_Group"
                hl.Adornee = alvo
                hl.FillColor = Color3.fromRGB(255, 215, 0)
                hl.FillTransparency = 0.3
                hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                hl.OutlineTransparency = 0
                hl.Parent = alvo
                ativoHighlight = hl

                -- 2. Cria o nome flutuante traduzido em cima do baú
                local billboard = Instance.new("BillboardGui")
                billboard.Name = "ChestNameGroup"
                billboard.Adornee = alvo
                billboard.Size = UDim2.new(0, 220, 0, 50)
                billboard.StudsOffset = Vector3.new(0, 3, 0)
                billboard.AlwaysOnTop = true
                billboard.Parent = alvo

                local textLabel = Instance.new("TextLabel")
                textLabel.Size = UDim2.new(1, 0, 1, 0)
                textLabel.BackgroundTransparency = 1
                textLabel.Text = nomeTipo
                textLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
                textLabel.TextStrokeTransparency = 0
                textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
                textLabel.Font = Enum.Font.SourceSansBold
                textLabel.TextSize = 16
                textLabel.Parent = billboard

                ativoBillboard = billboard
                btn.BackgroundColor3 = Color3.fromRGB(50, 120, 60)
                btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            end
        end)
    end

    if contador == 0 then
        local msg = Instance.new("TextButton")
        msg.Size = UDim2.new(1, -10, 0, 32)
        msg.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
        msg.Text = "Nenhum baú encontrado!"
        msg.TextColor3 = Color3.fromRGB(180, 180, 180)
        msg.Font = Enum.Font.SourceSans
        msg.TextSize = 14
        msg.Parent = ScrollContainer
    end
end

RefreshBtn.MouseButton1Click:Connect(AtualizarListaBaus)

--------------------------------------------------------------------------------
-- CONTROLE DA TECLA '1' E BOTÃO DE MINIMIZAR
--------------------------------------------------------------------------------
local function ToggleGUI()
    MainFrame.Visible = not MainFrame.Visible
end

CloseBtn.MouseButton1Click:Connect(ToggleGUI)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.One then
            ToggleGUI()
        end
    end
end)

-- Botão Flutuante (Mobile)
local OpenBtn = Instance.new("TextButton")
OpenBtn.Name = "OpenBtn"
OpenBtn.Size = UDim2.new(0, 45, 0, 45)
OpenBtn.Position = UDim2.new(0, 10, 0.4, 0)
OpenBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
OpenBtn.Text = "BAÚS"
OpenBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
OpenBtn.Font = Enum.Font.SourceSansBold
OpenBtn.TextSize = 12
OpenBtn.Parent = ScreenGui

local OpenCorner = Instance.new("UICorner")
OpenCorner.CornerRadius = UDim.new(1, 0)
OpenCorner.Parent = OpenBtn

OpenBtn.MouseButton1Click:Connect(ToggleGUI)

-- Carrega a lista automaticamente
AtualizarListaBaus()
