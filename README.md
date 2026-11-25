--[[
    PRIME X - SCRIPT HUB MOBILE
    AUTO CAIXA + EMPILHADOR + MINERADOR
    Interface adaptada para toque/celular.
]]

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local PathfindingService = game:GetService("PathfindingService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local VirtualUser = game:GetService("VirtualUser")

local Player = Players.LocalPlayer
local PlayerGui = Player:WaitForChild("PlayerGui")
-- Anti-AFK global: protege todas as rotinas enquanto o hub estiver carregado.
pcall(function()
    Player.Idled:Connect(function()
        pcall(function()
            VirtualUser:CaptureController()
            VirtualUser:ClickButton2(Vector2.new(0, 0))
        end)
    end)
end)
local env = (type(getgenv) == "function" and getgenv()) or _G

-- Encerra execucoes anteriores, se existirem.
if env.MaxiumHub then
    env.MaxiumHub.CaixaAtivo = false
    env.MaxiumHub.VarrerAtivo = false
end
if env.XLZMinerador then
    env.XLZMinerador.Running = false
    env.XLZMinerador.Mining = false
end
if env.AutoEmpilhadorState then
    env.AutoEmpilhadorState.Running = false
    env.AutoEmpilhadorState.Token = (env.AutoEmpilhadorState.Token or 0) + 1
end

local oldGui = PlayerGui:FindFirstChild("PRIMEXPanel")
if oldGui then oldGui:Destroy() end

local IMAGE_ID = "rbxassetid://106413301562079"
local RED = Color3.fromRGB(220, 35, 55) -- Vermelho PRIME X
local DARK = Color3.fromRGB(9, 9, 13)
local DARK2 = Color3.fromRGB(19, 19, 26)
local WHITE = Color3.fromRGB(245, 245, 250)
local OFF = Color3.fromRGB(35, 35, 35)
local GREEN = Color3.fromRGB(35, 145, 85)

local function tween(obj, duration, props)
    local anim = TweenService:Create(obj, TweenInfo.new(duration, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), props)
    anim:Play()
    return anim
end

local Hub = {
    CaixaAtivo = false,
    CaixaVelocidade = 24,
    Velocidade = 24,
    Movimento = nil
}
env.MaxiumHub = Hub

local Emp = {Running = false, Token = 0}
env.AutoEmpilhadorState = Emp

local function Personagem()
    local char = Player.Character
    if not char then return nil, nil end
    return char:FindFirstChild("HumanoidRootPart"), char:FindFirstChildOfClass("Humanoid")
end

local function AdquirirMovimento(nome, ativo)
    while ativo() and Hub.Movimento ~= nil do task.wait(0.15) end
    if not ativo() then return false end
    Hub.Movimento = nome
    return true
end

local function LiberarMovimento(nome)
    if Hub.Movimento == nome then Hub.Movimento = nil end
end

-- =========================================================
-- PAINEL PRINCIPAL
-- =========================================================
local Gui = Instance.new("ScreenGui")
Gui.Name = "PRIMEXPanel"
Gui.ResetOnSpawn = false
Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
Gui.Parent = PlayerGui

local Frame = Instance.new("Frame")
Frame.Name = "MainPanel"
Frame.Size = UDim2.fromOffset(650, 430)
Frame.Position = UDim2.new(0.5, 0, 0.5, 0)
Frame.AnchorPoint = Vector2.new(0.5, 0.5)
Frame.BackgroundColor3 = DARK
Frame.BorderSizePixel = 0
Frame.ClipsDescendants = true
Frame.Active = true
Frame.Parent = Gui
Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 12)

local PanelScale = Instance.new("UIScale")
local camera = workspace.CurrentCamera
local viewport = camera and camera.ViewportSize or Vector2.new(650, 430)
local escalaInicial = math.clamp(math.min(viewport.X / 680, viewport.Y / 460), 0.45, 1) * 100
PanelScale.Scale = escalaInicial / 100
PanelScale.Parent = Frame

local Stroke = Instance.new("UIStroke")
Stroke.Color = RED
Stroke.Thickness = 1.5
Stroke.Transparency = 0.08
Stroke.Parent = Frame

local Background = Instance.new("ImageLabel")
Background.Name = "HackerBackground"
Background.Size = UDim2.fromScale(1, 1)
Background.BackgroundTransparency = 1
Background.Image = IMAGE_ID
Background.ScaleType = Enum.ScaleType.Stretch
Background.ImageTransparency = 0.88
Background.ZIndex = 1
Background.Parent = Frame
Instance.new("UICorner", Background).CornerRadius = UDim.new(0, 12)

local Header = Instance.new("TextLabel")
Header.Size = UDim2.new(1, -130, 0, 42)
Header.Position = UDim2.fromOffset(15, 2)
Header.BackgroundTransparency = 1
Header.Text = "PRIME X  |  MOBILE"
Header.TextColor3 = WHITE
Header.TextSize = 18
Header.Font = Enum.Font.GothamBlack
Header.TextXAlignment = Enum.TextXAlignment.Left
Header.ZIndex = 3
Header.Parent = Frame

local DragArea = Instance.new("TextButton")
DragArea.Name = "DragArea"
DragArea.Size = UDim2.new(1, -115, 0, 45)
DragArea.BackgroundTransparency = 1
DragArea.Text = ""
DragArea.AutoButtonColor = false
DragArea.ZIndex = 4
DragArea.Parent = Frame

local ConfigButton = Instance.new("TextButton")
ConfigButton.Size = UDim2.fromOffset(38, 38)
ConfigButton.Position = UDim2.new(1, -88, 0, 8)
ConfigButton.BackgroundColor3 = DARK2
ConfigButton.Text = "⚙"
ConfigButton.TextColor3 = WHITE
ConfigButton.TextSize = 23
ConfigButton.Font = Enum.Font.GothamBold
ConfigButton.AutoButtonColor = false
ConfigButton.ZIndex = 5
ConfigButton.Parent = Frame
Instance.new("UICorner", ConfigButton).CornerRadius = UDim.new(0, 10)
local ConfigStroke = Instance.new("UIStroke", ConfigButton)
ConfigStroke.Color = RED
ConfigStroke.Transparency = 0.35
local ConfigGradient = Instance.new("UIGradient")
ConfigGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(220, 35, 55)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(75, 15, 24))
})
ConfigGradient.Rotation = 45
ConfigGradient.Parent = ConfigButton

local MinButton = Instance.new("TextButton")
MinButton.Size = UDim2.fromOffset(38, 38)
MinButton.Position = UDim2.new(1, -46, 0, 8)
MinButton.BackgroundColor3 = DARK2
MinButton.Text = "—"
MinButton.TextColor3 = WHITE
MinButton.TextSize = 19
MinButton.Font = Enum.Font.GothamBold
MinButton.AutoButtonColor = false
MinButton.ZIndex = 5
MinButton.Parent = Frame
Instance.new("UICorner", MinButton).CornerRadius = UDim.new(0, 10)
local MinStroke = Instance.new("UIStroke", MinButton)
MinStroke.Color = RED
MinStroke.Transparency = 0.35

local Bubble = Instance.new("TextButton")
Bubble.Name = "PRIMEXMenuButton"
Bubble.Size = UDim2.fromOffset(86, 38)
Bubble.Position = UDim2.new(1, -108, 1, -78)
Bubble.BackgroundColor3 = RED
Bubble.Text = "MENU"
Bubble.TextColor3 = WHITE
Bubble.TextSize = 14
Bubble.Font = Enum.Font.GothamBlack
Bubble.AutoButtonColor = false
Bubble.Visible = false
Bubble.ZIndex = 10
Bubble.Parent = Gui
Instance.new("UICorner", Bubble).CornerRadius = UDim.new(0, 9)
local BubbleStroke = Instance.new("UIStroke", Bubble)
BubbleStroke.Color = Color3.fromRGB(255, 105, 115)
BubbleStroke.Thickness = 1.5
local BubbleScale = Instance.new("UIScale", Bubble)
Bubble.MouseEnter:Connect(function()
    tween(Bubble, 0.12, {BackgroundColor3 = Color3.fromRGB(245, 55, 70)})
end)
Bubble.MouseLeave:Connect(function()
    tween(Bubble, 0.12, {BackgroundColor3 = RED})
end)

-- Navegacao lateral + paginas independentes (layout PRIME X)
local Sidebar = Instance.new("Frame")
Sidebar.Name = "Sidebar"
Sidebar.Position = UDim2.fromOffset(10, 54)
Sidebar.Size = UDim2.new(0, 142, 1, -66)
Sidebar.BackgroundColor3 = Color3.fromRGB(13, 12, 20)
Sidebar.BorderSizePixel = 0
Sidebar.ZIndex = 3
Sidebar.Parent = Frame
Instance.new("UICorner", Sidebar).CornerRadius = UDim.new(0, 10)

local SideTitle = Instance.new("TextLabel")
SideTitle.Size = UDim2.new(1, -18, 0, 34)
SideTitle.Position = UDim2.fromOffset(10, 8)
SideTitle.BackgroundTransparency = 1
SideTitle.Text = "PX  •  MENU"
SideTitle.TextColor3 = RED
SideTitle.Font = Enum.Font.GothamBlack
SideTitle.TextSize = 13
SideTitle.TextXAlignment = Enum.TextXAlignment.Left
SideTitle.ZIndex = 5
SideTitle.Parent = Sidebar

local Pages = {}
local PageButtons = {}
local pageNames = {"AUTO CAIXA", "EMPILHADOR", "MINERADOR"}
local pageIcons = {"▣", "▤", "⛏"}
local activePage = 1

for i, pageName in ipairs(pageNames) do
    local page = Instance.new("ScrollingFrame")
    page.Name = "Page" .. i
    page.Position = UDim2.fromOffset(164, 54)
    page.Size = UDim2.new(1, -176, 1, -66)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 3
    page.ScrollBarImageColor3 = RED
    page.CanvasSize = UDim2.new()
    page.Visible = (i == 1)
    page.ZIndex = 3
    page.Parent = Frame
    Pages[i] = page

    local pageLayout = Instance.new("UIListLayout")
    pageLayout.Padding = UDim.new(0, 10)
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Parent = page
    pageLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.fromOffset(0, pageLayout.AbsoluteContentSize.Y + 14)
    end)

    local nav = Instance.new("TextButton")
    nav.Name = "Nav" .. i
    nav.Position = UDim2.fromOffset(7, 52 + (i - 1) * 48)
    nav.Size = UDim2.new(1, -14, 0, 40)
    nav.BackgroundColor3 = (i == 1) and Color3.fromRGB(78, 22, 32) or Color3.fromRGB(20, 18, 28)
    nav.BorderSizePixel = 0
    nav.Text = pageIcons[i] .. "   " .. pageName
    nav.TextColor3 = WHITE
    nav.TextSize = 10
    nav.Font = Enum.Font.GothamBold
    nav.TextXAlignment = Enum.TextXAlignment.Left
    nav.AutoButtonColor = false
    nav.ZIndex = 5
    nav.Parent = Sidebar
    Instance.new("UICorner", nav).CornerRadius = UDim.new(0, 8)
    local navPad = Instance.new("UIPadding", nav)
    navPad.PaddingLeft = UDim.new(0, 10)
    local navStroke = Instance.new("UIStroke", nav)
    navStroke.Color = RED
    navStroke.Transparency = (i == 1) and 0.25 or 0.88
    PageButtons[i] = nav

    nav.MouseButton1Click:Connect(function()
        activePage = i
        for j, otherPage in ipairs(Pages) do
            otherPage.Visible = (j == i)
            PageButtons[j].BackgroundColor3 = (j == i) and Color3.fromRGB(78, 22, 32) or Color3.fromRGB(20, 18, 28)
            local stroke = PageButtons[j]:FindFirstChildOfClass("UIStroke")
            if stroke then stroke.Transparency = (j == i) and 0.25 or 0.88 end
        end
    end)
end

local Conteudo = Pages[1] -- compatibilidade com a estrutura das secoes

local function makeLabel(parent, text, y, h, size)
    local l = Instance.new("TextLabel")
    l.Position = UDim2.fromOffset(12, y)
    l.Size = UDim2.new(1, -24, 0, h)
    l.BackgroundTransparency = 1
    l.Text = text
    l.TextColor3 = WHITE
    l.Font = Enum.Font.GothamMedium
    l.TextSize = size or 12
    l.TextXAlignment = Enum.TextXAlignment.Left
    l.TextWrapped = true
    l.ZIndex = 5
    l.Parent = parent
    return l
end

local function makeButton(parent, text, x, y, w, h, color)
    local b = Instance.new("TextButton")
    b.Position = UDim2.fromOffset(x, y)
    b.Size = UDim2.new(w, 0, 0, h)
    b.BackgroundColor3 = color or OFF
    b.Text = text
    b.TextColor3 = WHITE
    b.Font = Enum.Font.GothamBold
    b.TextSize = 11
    b.AutoButtonColor = false
    b.ZIndex = 6
    b.Parent = parent
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 7)
    local s = Instance.new("UIStroke", b)
    s.Color = Color3.fromRGB(65, 65, 65)
    s.Transparency = 0.25
    b.MouseEnter:Connect(function()
        tween(b, 0.12, {BackgroundColor3 = Color3.fromRGB(105, 25, 38)})
    end)
    b.MouseLeave:Connect(function()
        local active = string.find(string.upper(b.Text), "DESATIVAR") ~= nil
        tween(b, 0.12, {BackgroundColor3 = active and RED or (color or OFF)})
    end)
    return b
end

local function setButtonActive(button, active, onText, offText)
    button.Text = active and onText or offText
    tween(button, 0.15, {BackgroundColor3 = active and RED or OFF})
end

-- Secoes recolhiveis
local function CriarSecao(nome, ordem, alturaFechada, alturaAberta)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(1, -4, 0, alturaFechada)
    card.BackgroundColor3 = Color3.fromRGB(12, 12, 17)
    card.BorderSizePixel = 0
    card.ClipsDescendants = true
    card.LayoutOrder = ordem
    card.ZIndex = 4
    card.Parent = Pages[ordem] or Pages[1]
    Instance.new("UICorner", card).CornerRadius = UDim.new(0, 9)
    local cs = Instance.new("UIStroke", card)
    cs.Color = Color3.fromRGB(125, 35, 48)
    cs.Transparency = 0.35

    local header = Instance.new("TextButton")
    header.Size = UDim2.new(1, 0, 0, 42)
    header.BackgroundColor3 = Color3.fromRGB(20, 16, 21)
    header.Text = "⌄   " .. nome
    header.TextColor3 = WHITE
    header.TextSize = 14
    header.Font = Enum.Font.GothamBlack
    header.TextXAlignment = Enum.TextXAlignment.Left
    header.AutoButtonColor = false
    header.ZIndex = 6
    header.Parent = card
    Instance.new("UICorner", header).CornerRadius = UDim.new(0, 8)
    local pad = Instance.new("UIPadding", header)
    pad.PaddingLeft = UDim.new(0, 14)

    local body = Instance.new("Frame")
    body.Position = UDim2.fromOffset(0, 44)
    body.Size = UDim2.new(1, 0, 0, alturaAberta - 44)
    body.BackgroundTransparency = 1
    body.Visible = false
    body.ZIndex = 5
    body.Parent = card

    local aberto = false
    header.MouseButton1Click:Connect(function()
        aberto = not aberto
        header.Text = (aberto and "⌃   " or "⌄   ") .. nome
        body.Visible = aberto
        tween(card, 0.18, {Size = UDim2.new(1, -4, 0, aberto and alturaAberta or alturaFechada)})
    end)
    return body
end

-- =========================================================
-- CONFIGURACOES / SLIDER DE ESCALA
-- =========================================================
local escalaAtual = 100
local sliderDragging = false
local Settings = Instance.new("Frame")
Settings.Size = UDim2.fromOffset(310, 205)
Settings.Position = UDim2.new(0.5, -155, 0.5, -102)
Settings.BackgroundColor3 = Color3.fromRGB(15, 13, 23)
Settings.BackgroundTransparency = 0.05
Settings.BorderSizePixel = 0
Settings.Visible = false
Settings.ZIndex = 20
Settings.Parent = Gui
Instance.new("UICorner", Settings).CornerRadius = UDim.new(0, 12)
local ss = Instance.new("UIStroke", Settings)
ss.Color = RED
ss.Thickness = 2
local SettingsScale = Instance.new("UIScale", Settings)
SettingsScale.Scale = 0.8

local st = Instance.new("TextLabel")
st.Size = UDim2.new(1, -55, 0, 35)
st.Position = UDim2.fromOffset(14, 8)
st.BackgroundTransparency = 1
st.Text = "⚙   PRIME X  /  CONFIGURAÇÕES"
st.TextColor3 = WHITE
st.TextSize = 14
st.Font = Enum.Font.GothamBold
st.TextXAlignment = Enum.TextXAlignment.Left
st.ZIndex = 21
st.Parent = Settings

local close = Instance.new("TextButton")
close.Size = UDim2.fromOffset(30, 30)
close.Position = UDim2.new(1, -38, 0, 8)
close.BackgroundColor3 = Color3.fromRGB(105, 25, 38)
close.Text = "✕"
close.TextColor3 = WHITE
close.TextSize = 18
close.Font = Enum.Font.GothamBold
close.ZIndex = 21
close.Parent = Settings
Instance.new("UICorner", close).CornerRadius = UDim.new(0, 8)

local div = Instance.new("Frame")
div.Size = UDim2.new(1, -26, 0, 2)
div.Position = UDim2.fromOffset(13, 48)
div.BackgroundColor3 = RED
div.BorderSizePixel = 0
div.ZIndex = 21
div.Parent = Settings

local scaleTitle = Instance.new("TextLabel")
scaleTitle.Size = UDim2.new(1, -30, 0, 25)
scaleTitle.Position = UDim2.fromOffset(15, 61)
scaleTitle.BackgroundTransparency = 1
scaleTitle.Text = "TAMANHO DA INTERFACE"
scaleTitle.TextColor3 = WHITE
scaleTitle.TextSize = 11
scaleTitle.Font = Enum.Font.GothamBold
scaleTitle.TextXAlignment = Enum.TextXAlignment.Left
scaleTitle.ZIndex = 21
scaleTitle.Parent = Settings

local scaleValue = Instance.new("TextLabel")
scaleValue.Size = UDim2.fromOffset(55, 25)
scaleValue.Position = UDim2.new(1, -70, 0, 61)
scaleValue.BackgroundTransparency = 1
scaleValue.Text = "100%"
scaleValue.TextColor3 = RED
scaleValue.TextSize = 12
scaleValue.Font = Enum.Font.GothamBold
scaleValue.ZIndex = 21
scaleValue.Parent = Settings

local Track = Instance.new("TextButton")
Track.Size = UDim2.new(1, -36, 0, 8)
Track.Position = UDim2.fromOffset(18, 105)
Track.BackgroundColor3 = Color3.fromRGB(55, 55, 65)
Track.BorderSizePixel = 0
Track.Text = ""
Track.ZIndex = 22
Track.Parent = Settings
Instance.new("UICorner", Track).CornerRadius = UDim.new(1, 0)

local Fill = Instance.new("Frame")
Fill.Size = UDim2.new((escalaAtual - 10) / 190, 0, 1, 0)
Fill.BackgroundColor3 = RED
Fill.BorderSizePixel = 0
Fill.ZIndex = 23
Fill.Parent = Track
Instance.new("UICorner", Fill).CornerRadius = UDim.new(1, 0)

local Knob = Instance.new("Frame")
Knob.Size = UDim2.fromOffset(16, 16)
Knob.AnchorPoint = Vector2.new(0.5, 0.5)
Knob.Position = UDim2.new((escalaAtual - 10) / 190, 0, 0.5, 0)
Knob.BackgroundColor3 = WHITE
Knob.BorderSizePixel = 0
Knob.ZIndex = 24
Knob.Parent = Track
Instance.new("UICorner", Knob).CornerRadius = UDim.new(1, 0)
local ks = Instance.new("UIStroke", Knob)
ks.Color = RED
ks.Thickness = 2

local function atualizarEscala(valor)
    escalaAtual = math.clamp(math.floor(valor + 0.5), 10, 200)
    PanelScale.Scale = escalaAtual / 100
    scaleValue.Text = escalaAtual .. "%"
    local p = (escalaAtual - 10) / 190
    Fill.Size = UDim2.new(p, 0, 1, 0)
    Knob.Position = UDim2.new(p, 0, 0.5, 0)
end

local function atualizarSlider(x)
    local inicio, largura = Track.AbsolutePosition.X, Track.AbsoluteSize.X
    if largura <= 0 then return end
    local p = math.clamp((x - inicio) / largura, 0, 1)
    atualizarEscala(10 + p * 190)
end

Track.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = true
        atualizarSlider(input.Position.X)
    end
end)
UIS.InputChanged:Connect(function(input)
    if sliderDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        atualizarSlider(input.Position.X)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        sliderDragging = false
    end
end)

local configAberta = false
local function abrirConfiguracoes()
    configAberta = not configAberta
    if configAberta then
        Settings.Visible = true
        Settings.BackgroundTransparency = 1
        SettingsScale.Scale = 0.8
        tween(SettingsScale, 0.3, {Scale = 1})
        tween(Settings, 0.3, {BackgroundTransparency = 0.05, Position = UDim2.new(0.5, -145, 0.5, -95)})
    else
        tween(SettingsScale, 0.15, {Scale = 0.8})
        tween(Settings, 0.15, {BackgroundTransparency = 1})
        task.delay(0.16, function() if not configAberta then Settings.Visible = false end end)
    end
end
ConfigButton.MouseButton1Click:Connect(abrirConfiguracoes)
close.MouseButton1Click:Connect(abrirConfiguracoes)

-- Minimizar / restaurar
local minimizado = false
local function minimizarPainel()
    minimizado = true
    configAberta = false
    Settings.Visible = false
    Frame.Visible = false
    Bubble.Visible = true
    BubbleScale.Scale = 0.5
    tween(BubbleScale, 0.3, {Scale = 1})
end
local function restaurarPainel()
    minimizado = false
    Bubble.Visible = false
    Frame.Visible = true
    PanelScale.Scale = escalaAtual / 100
end
MinButton.MouseButton1Click:Connect(minimizarPainel)

-- Arrastar painel e bolinha
local dragging, dragStart, startPos = false, nil, nil
DragArea.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = Frame.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then dragging = false end
end)

local bubbleDrag, bubbleStart, bubblePos, bubbleMoved = false, nil, nil, false
Bubble.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        bubbleDrag = true
        bubbleMoved = false
        bubbleStart = input.Position
        bubblePos = Bubble.Position
    end
end)
UIS.InputChanged:Connect(function(input)
    if bubbleDrag and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - bubbleStart
        if delta.Magnitude > 5 then bubbleMoved = true end
        Bubble.Position = UDim2.new(bubblePos.X.Scale, bubblePos.X.Offset + delta.X, bubblePos.Y.Scale, bubblePos.Y.Offset + delta.Y)
    end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        bubbleDrag = false
    end
end)
Bubble.MouseButton1Click:Connect(function()
    if not bubbleMoved then restaurarPainel() end
end)

-- =========================================================
-- AUTO MINERADOR INTEGRADO | versao mobile
-- =========================================================
--[[
    XLZ HUB | AUTO MINERADOR
    - Exige que a picareta ja esteja equipada ao iniciar
    - Depois da venda, pressiona 1 para selecionar o primeiro slot
    - Remove o ponto A e percorre os quatro pontos restantes
    - Voo suave ate os pontos, com estabilizacao ao chegar
    - Aproxima do minerio e aguarda o prompt ficar disponivel
    - Mantem a orientacao final de cada destino
    - Cooldown individual de 20 segundos por minerio
    - Vende automaticamente a cada 9 minerios (90 de inventario)
    - Retorna e continua minerando sem desligar o painel
    - Status integrado ao painel principal, com indicador verde quando ativo
]]

local ProximityPromptService = game:GetService("ProximityPromptService")
local VirtualInputManager = game:GetService("VirtualInputManager")

if env.XLZMinerador then
    env.XLZMinerador.Running = false
    env.XLZMinerador.Mining = false
end

local State = {
    Running = true, Mining = false, Speed = 16,
    HoldTime = 16, OreCooldown = 20, LastOre = {},
    Status = "Aguardando inicio", Flying = false, PointIndex = 1,
    VisiblePrompts = {}
}
env.XLZMinerador = State

local ORES_POR_VENDA = 9
local MineriosDesdeVenda = 0

-- Detecta o momento exato em que o prompt aparece na tela do jogador.
ProximityPromptService.PromptShown:Connect(function(prompt)
    State.VisiblePrompts[prompt] = true
end)
ProximityPromptService.PromptHidden:Connect(function(prompt)
    State.VisiblePrompts[prompt] = nil
end)

-- Ponto A removido. Ordem fixa: antigos pontos 2 -> 3 -> 4 -> 5 -> repetir.
-- A rotacao dos CFrames mantem o personagem reto e virado conforme cada local.
local Points = {
    CFrame.new(354.671967, -0.109608, 2031.210693, -0.971223, 0, 0.238173, 0, 1, 0, -0.238173, 0, -0.971223),
    CFrame.new(354.370209, 1.051181, 2002.388550, 0.873217, 0, 0.487332, 0, 1, 0, -0.487332, 0, 0.873217),
    CFrame.new(409.504974, 4.854165, 1993.255737, 0.970323, 0, -0.241813, 0, 1, 0, 0.241813, 0, 0.970323),
    CFrame.new(408.488556, 7.248827, 2028.808472, -0.135797, 0, -0.990737, 0, 1, 0, 0.990737, 0, -0.135797)
}

local Character, Humanoid, Root

local function refreshCharacter()
    Character = Player.Character or Player.CharacterAdded:Wait()
    Humanoid = Character:WaitForChild("Humanoid")
    Root = Character:WaitForChild("HumanoidRootPart")
end

refreshCharacter()

local MiningCameraConnection = nil
local SavedCameraType, SavedCameraSubject = nil, nil
local SavedMinZoom, SavedMaxZoom = nil, nil

local function pararCameraMineracao(restaurar)
    if MiningCameraConnection then
        MiningCameraConnection:Disconnect()
        MiningCameraConnection = nil
    end
    local camera = Workspace.CurrentCamera
    if restaurar and camera then
        camera.CameraType = SavedCameraType or Enum.CameraType.Custom
        camera.CameraSubject = SavedCameraSubject or Humanoid
        if SavedMinZoom then Player.CameraMinZoomDistance = SavedMinZoom end
        if SavedMaxZoom then Player.CameraMaxZoomDistance = SavedMaxZoom end
    end
end

local function iniciarCameraSuperior()
    local camera = Workspace.CurrentCamera
    if not camera then return end
    if not SavedCameraType then
        SavedCameraType = camera.CameraType
        SavedCameraSubject = camera.CameraSubject
        SavedMinZoom = Player.CameraMinZoomDistance
        SavedMaxZoom = Player.CameraMaxZoomDistance
    end
    pararCameraMineracao(false)
    camera.CameraType = Enum.CameraType.Scriptable
    MiningCameraConnection = RunService.RenderStepped:Connect(function()
        local root = Root
        if State.Running and State.Mining and root and root.Parent then
            local foco = root.Position + Vector3.new(0, 1.5, 0)
            camera.CFrame = CFrame.lookAt(root.Position + Vector3.new(0, 15, 0), foco)
            camera.Focus = CFrame.new(foco)
        end
    end)
end

local function enquadrarCameraNoNpc(parteNpc)
    pararCameraMineracao(false)
    local camera = Workspace.CurrentCamera
    local character = Player.Character
    local root = character and character:FindFirstChild("HumanoidRootPart")
    if not camera or not character or not root or not parteNpc or not parteNpc.Parent then return end
    camera.CameraType = Enum.CameraType.Scriptable
    local direcao = root.Position - parteNpc.Position
    direcao = Vector3.new(direcao.X, 0, direcao.Z)
    if direcao.Magnitude < 0.1 then direcao = Vector3.new(0, 0, 1) end
    direcao = direcao.Unit
    local foco = parteNpc.Position + Vector3.new(0, 2.2, 0)
    local posCamera = root.Position + direcao * 13 + Vector3.new(0, 6, 0)
    camera.CFrame = CFrame.lookAt(posCamera, foco)
    camera.Focus = CFrame.new(foco)
    Player.CameraMinZoomDistance = 12
    Player.CameraMaxZoomDistance = 18
end

local function voltarCameraSuperior()
    if State.Running and State.Mining then
        iniciarCameraSuperior()
    else
        pararCameraMineracao(true)
    end
end

local stopFlight
Player.CharacterAdded:Connect(function()
    if stopFlight then stopFlight() end
    OriginalCollisions = {}
    task.wait(0.75)
    if State.Running then
        refreshCharacter()
        State.Flying = false
    end
end)

local function getChild(parent, name, timeout)
    if not parent then return nil end
    local ok, result = pcall(function()
        return parent:WaitForChild(name, timeout or 8)
    end)
    return ok and result or nil
end

local Construcoes = getChild(workspace, "Construcoes", 8)
local MineFolder = getChild(Construcoes, "Minerador", 8)

local PickPrompt
if MineFolder then
    -- Primeiro tenta a hierarquia original; depois procura o prompt por toda a pasta.
    local pickFolder = MineFolder:FindFirstChild("PegarPicareta", true)
    local pickBlock = pickFolder and pickFolder:FindFirstChild("ProximityBlock", true)
    PickPrompt = pickBlock and pickBlock:FindFirstChildWhichIsA("ProximityPrompt", true)

    if not PickPrompt then
        for _, obj in ipairs(MineFolder:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                local label = string.lower((obj.ActionText or "") .. " " .. (obj.ObjectText or "") .. " " .. obj:GetFullName())
                if string.find(label, "picareta", 1, true) or string.find(label, "pickaxe", 1, true) then
                    PickPrompt = obj
                    break
                end
            end
        end
    end
end

if not MineFolder then
    warn("[XLZ] Nao encontrei Workspace.Construcoes.Minerador.")
elseif not PickPrompt then
    warn("[XLZ] Pasta Minerador encontrada, mas nenhum prompt de picareta foi identificado.")
end

-- ProximityPrompt pode estar dentro de um Attachment. Nesse caso, a peca
-- fica acima dele na hierarquia, nao dentro dele.
local function getPromptPart(prompt)
    if not prompt or not prompt.Parent then return nil end

    -- Prompt pode estar diretamente numa BasePart...
    local current = prompt.Parent
    while current and current ~= workspace do
        if current:IsA("BasePart") then
            return current
        end

        -- ...ou dentro de um Model chamado Pedra. Nesse caso, procura
        -- a peca mais proxima do personagem em vez de retornar nil.
        if current:IsA("Model") then
            if current.PrimaryPart then
                return current.PrimaryPart
            end

            local character = Player.Character
            local root = character and character:FindFirstChild("HumanoidRootPart")
            local bestPart, bestDistance = nil, math.huge
            for _, descendant in ipairs(current:GetDescendants()) do
                if descendant:IsA("BasePart") then
                    local distance = root and (descendant.Position - root.Position).Magnitude or 0
                    if distance < bestDistance then
                        bestPart, bestDistance = descendant, distance
                    end
                end
            end
            if bestPart then return bestPart end
        end
        current = current.Parent
    end
    return nil
end

local function horizontalDistance(a, b)
    local dx, dz = a.X - b.X, a.Z - b.Z
    return math.sqrt(dx * dx + dz * dz)
end

local function isMiningPrompt(prompt)
    if not prompt or not prompt:IsA("ProximityPrompt") then return false end
    local action = string.lower(prompt.ActionText or "")
    local object = string.lower(prompt.ObjectText or "")
    return string.find(action, "miner", 1, true) ~= nil
        or string.find(action, "mine", 1, true) ~= nil
        or string.find(object, "miner", 1, true) ~= nil
        or string.find(object, "mine", 1, true) ~= nil
end

local function getHumanoidRoot()
    if not Character or not Character.Parent or not Humanoid or not Humanoid.Parent
        or not Root or not Root.Parent then
        refreshCharacter()
    end
    return Humanoid, Root
end

-- Confere se a picareta ja esta na mao antes de iniciar a mineracao.
local function hasEquippedPickaxe()
    if not Character then return false end
    for _, item in ipairs(Character:GetChildren()) do
        if item:IsA("Tool") then
            local n = string.lower(item.Name)
            if string.find(n, "picareta", 1, true) or string.find(n, "pickaxe", 1, true) then
                return true
            end
        end
    end
    return false
end

-- Caminhada normal com Pathfinding: velocidade ajustavel, curvas suaves
-- e tentativa de pulo quando detectar que o personagem ficou preso.
local FlightVelocity, FlightGyro
local NoclipConnection
local OriginalCollisions = {}
local SavedAutoRotate

-- Atravessa somente paredes/pecas com o proprio personagem enquanto ativo.
-- Os valores originais de CanCollide sao restaurados ao desativar.
local function setCharacterNoclip(enabled)
    if enabled then
        if NoclipConnection then return end
        local function applyNoClip()
            if not Character or not Character.Parent then return end
            for _, part in ipairs(Character:GetDescendants()) do
                if part:IsA("BasePart") then
                    if OriginalCollisions[part] == nil then
                        OriginalCollisions[part] = part.CanCollide
                    end
                    part.CanCollide = false
                end
            end
        end
        applyNoClip()
        NoclipConnection = RunService.Stepped:Connect(applyNoClip)
    else
        if NoclipConnection then
            NoclipConnection:Disconnect()
            NoclipConnection = nil
        end
        for part, wasCollidable in pairs(OriginalCollisions) do
            if part and part.Parent then
                pcall(function() part.CanCollide = wasCollidable end)
            end
        end
        table.clear(OriginalCollisions)
    end
end

stopFlight = function()
    -- Desliga primeiro o noclip e devolve as colisoes originais.
    setCharacterNoclip(false)
    if FlightVelocity then pcall(function() FlightVelocity:Destroy() end) end
    if FlightGyro then pcall(function() FlightGyro:Destroy() end) end
    FlightVelocity, FlightGyro = nil, nil
    State.Flying = false

    -- Recupera o controle normal do personagem ao desligar.
    local savedAutoRotate = SavedAutoRotate
    SavedAutoRotate = nil
    local hum = Humanoid
    local root = Root
    if hum and hum.Parent then
        pcall(function()
            hum.PlatformStand = false
            hum.Sit = false
            hum.AutoRotate = (savedAutoRotate == nil and true or savedAutoRotate)
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
            task.delay(0.15, function()
                if hum.Parent and not State.Mining then
                    hum:ChangeState(Enum.HumanoidStateType.Running)
                end
            end)
        end)
    end
    if root and root.Parent then
        pcall(function()
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
        end)
    end
end

local function getFlightControl(root)
    setCharacterNoclip(true)
    if Humanoid and SavedAutoRotate == nil then SavedAutoRotate = Humanoid.AutoRotate end
    if not FlightVelocity or not FlightVelocity.Parent then
        FlightVelocity = Instance.new("BodyVelocity")
        FlightVelocity.Name = "XLZ_FlightVelocity"
        FlightVelocity.MaxForce = Vector3.new(1e9, 1e9, 1e9)
        FlightVelocity.P = 50000
        FlightVelocity.Velocity = Vector3.zero
        FlightVelocity.Parent = root
    end
    if not FlightGyro or not FlightGyro.Parent then
        FlightGyro = Instance.new("BodyGyro")
        FlightGyro.Name = "XLZ_FlightGyro"
        FlightGyro.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
        FlightGyro.P = 50000
        FlightGyro.D = 1500
        FlightGyro.CFrame = root.CFrame
        FlightGyro.Parent = root
    end
    State.Flying = true
    return FlightVelocity, FlightGyro
end

local function holdPosition(rotation)
    local hum, root = getHumanoidRoot()
    if not root or not hum then return false end
    local bv, bg = getFlightControl(root)
    bv.Velocity = Vector3.zero
    if rotation then bg.CFrame = CFrame.new(root.Position) * rotation end
    hum:ChangeState(Enum.HumanoidStateType.Physics)
    State.Status = "Posicao estabilizada; aguardando interacao"
    return true
end

-- Voo suave em linha direta. O controle permanece ativo ao chegar,
-- mantendo o personagem estabilizado no ponto ate a proxima tarefa.
local function walkTo(targetCFrame, stopWhenPrompt)
    if not State.Running or not State.Mining then return false end
    local hum, root = getHumanoidRoot()
    if not hum or hum.Health <= 0 or not root then return false end

    local targetPosition = typeof(targetCFrame) == "CFrame" and targetCFrame.Position or targetCFrame
    local targetRotation = typeof(targetCFrame) == "CFrame" and targetCFrame.Rotation or root.CFrame.Rotation
    local bv, bg = getFlightControl(root)
    local started = os.clock()
    local lastPosition = root.Position
    local lastProgress = os.clock()
    local maxTime = math.max(12, (root.Position - targetPosition).Magnitude / math.max(State.Speed, 8) * 3 + 12)
    local reached = false

    hum.AutoRotate = false
    hum:ChangeState(Enum.HumanoidStateType.Physics)
    State.Status = "Voando ate o ponto " .. tostring(State.PointIndex)

    while State.Running and State.Mining and hum.Health > 0 and os.clock() - started < maxTime do
        -- Durante a aproximacao, interrompe o voo no instante em que o
        -- ProximityPrompt alvo fica visivel na tela.
        if stopWhenPrompt and State.VisiblePrompts[stopWhenPrompt] then
            reached = true
            break
        end

        local delta = targetPosition - root.Position
        local distance = delta.Magnitude
        bg.CFrame = CFrame.new(root.Position) * targetRotation

        if distance <= 2.0 then
            reached = true
            break
        end

        local speed = math.clamp(State.Speed * 2.5, 30, 85)
        local velocity = delta.Unit * math.min(speed, math.max(8, distance * 3))
        bv.Velocity = velocity

        if (root.Position - lastPosition).Magnitude > 0.5 then
            lastPosition = root.Position
            lastProgress = os.clock()
        elseif os.clock() - lastProgress > 2.5 then
            -- Pequeno impulso vertical para sair de enroscos.
            bv.Velocity = velocity + Vector3.new(0, 18, 0)
            lastProgress = os.clock()
            State.Status = "Ajustando voo para sair do obstaculo"
        end
        task.wait(0.05)
    end

    if not State.Running or not State.Mining then
        stopFlight()
        return false
    end

    if reached then
        bv.Velocity = Vector3.zero
        bg.CFrame = CFrame.new(root.Position) * targetRotation
        State.Status = "Chegou e ficou estabilizado no ponto " .. tostring(State.PointIndex)
        return true
    end

    bv.Velocity = Vector3.zero
    State.Status = "Nao alcancei o destino; confira as coordenadas"
    warn("[XLZ] Falha no voo. Atual:", root.Position, "Alvo:", targetPosition)
    return false
end

local function getOreKey(prompt, stones)
    local current = prompt and prompt.Parent
    local nearestModel = nil
    while current and current ~= stones do
        if current:IsA("Model") then
            nearestModel = current
            break
        end
        current = current.Parent
    end
    return nearestModel or prompt
end

local function findNearestMiningPrompt()
    if not MineFolder then return nil end
    local stones = MineFolder:FindFirstChild("Pedras")
    if not stones then return nil end
    local _, root = getHumanoidRoot()
    if not root then return nil end

    local bestPrompt, bestDistance = nil, math.huge
    local now = os.clock()
    for _, item in ipairs(stones:GetDescendants()) do
        if isMiningPrompt(item) and item.Enabled then
            local part = getPromptPart(item)
            local oreKey = getOreKey(item, stones)
            local last = State.LastOre[oreKey] or -math.huge
            if part and now - last >= State.OreCooldown then
                local distance = (part.Position - root.Position).Magnitude
                if distance < bestDistance then
                    bestPrompt, bestDistance = item, distance
                end
            end
        end
    end
    return bestPrompt, bestDistance
end

local function holdMiningPrompt(prompt)
    if not prompt or not prompt.Parent then return false end

    local began = false
    local completedFullHold = false
    local ok, err = pcall(function()
        prompt:InputHoldBegin()
        began = true

        -- Garante pelo menos 16 segundos reais de retenção.
        -- Se o próprio prompt exigir mais tempo, respeita o HoldDuration dele.
        local required = math.max(State.HoldTime, tonumber(prompt.HoldDuration) or 0)
        local started = os.clock()
        local lastStatusUpdate = 0

        while os.clock() - started < required do
            -- Se o jogador desligar a mineração, libera o botão imediatamente.
            if not State.Running or not State.Mining then
                break
            end
            if not prompt.Parent then
                break
            end

            local elapsed = os.clock() - started
            if elapsed - lastStatusUpdate >= 0.5 then
                State.Status = "Segurando mineracao: "
                    .. tostring(math.min(math.floor(elapsed), required))
                    .. "/" .. tostring(required) .. " segundos"
                lastStatusUpdate = elapsed
            end

            task.wait(0.05)
        end

        completedFullHold = (os.clock() - started) >= required
    end)

    if began then
        pcall(function()
            prompt:InputHoldEnd()
        end)
    end

    if not ok then
        warn("[PRIME X] Erro durante a retencao do prompt: " .. tostring(err))
        return false
    end

    -- Não contar como minério coletado se a retenção foi interrompida antes de 16s.
    return began and completedFullHold and State.Running and State.Mining
end

local function pressionarTecla1ParaEquiparPicareta()
    -- Depois da venda, pressiona 1 para selecionar o primeiro slot.
    local inputManager = game:GetService("VirtualInputManager")
    local ok, err = pcall(function()
        inputManager:SendKeyEvent(true, Enum.KeyCode.One, false, game)
        task.wait(0.08)
        inputManager:SendKeyEvent(false, Enum.KeyCode.One, false, game)
    end)
    if not ok then
        warn("[XLZ] Nao foi possivel pressionar 1 para equipar a picareta: " .. tostring(err))
        return false
    end
    task.wait(0.25)
    return true
end

local function activatePickaxe()
    -- Depois da venda, apenas pressiona 1 para selecionar a picareta.
    -- Nao procura, caminha ate ela nem interage com prompt algum.
    State.Status = "Selecionando picareta pelo slot 1"
    return pressionarTecla1ParaEquiparPicareta()
end

local function findMiningPromptAtPoint(center, radius)
    if not MineFolder then return nil end
    local stones = MineFolder:FindFirstChild("Pedras")
    if not stones then return nil end
    local bestPrompt, bestDistance = nil, radius or 18
    local now = os.clock()
    for _, item in ipairs(stones:GetDescendants()) do
        if isMiningPrompt(item) and item.Enabled then
            local part = getPromptPart(item)
            local key = getOreKey(item, stones)
            local last = State.LastOre[key] or -math.huge
            if part and now - last >= State.OreCooldown then
                local distance = horizontalDistance(part.Position, center)
                if distance <= bestDistance then
                    bestPrompt, bestDistance = item, distance
                end
            end
        end
    end
    return bestPrompt
end

local player = Player

local CONFIG = {
	-- Coordenadas de venda ficam somente na configuracao interna; nao aparecem no painel.
	TextoPromptAlvo = "vender minérios",
	TituloMenuAlvo = "companhia de mineração",
	CoordenadasCelular = {
		{ x = 462, y = 500 }, -- 1o Vender (500)
		{ x = 566, y = 504 }, -- 2o Vender (750)
		{ x = 682, y = 502 }, -- 3o Vender (1K)
		{ x = 790, y = 506 }, -- 4o Vender (6K)
		{ x = 886, y = 506 }, -- 5o Vender (25K)
	},
	CoordenadaFecharX = { x = 952, y = 256 },
	IntervaloCliqueVender = 1.0, -- intervalo entre cliques em "Vender"
	TempoExtraSegurar = 0.55, -- +0.5s (metade de 1s) extra segurando para abrir a venda
	TempoMaximoSegurando = 2.2,
	TempoEsperaMenu = 0.35,
	RaioDetectarCarro = 300,
	PosicaoVenda = Vector3.new(2933.27, 4.13, 3183.48),
	AlturaLevantadaTp = 18,
	EsperaCarregarMapa = 0.35,
	PosicaoRetorno = Vector3.new(386.17, 5.08, 2121.79),
	VelocidadeAproximacaoNpc = 18,
	DistanciaParadaNpc = 3.5,
}

local function obterPersonagem()
	local character = player.Character or player.CharacterAdded:Wait()
	local hrp = character:FindFirstChild("HumanoidRootPart")
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	return character, hrp, humanoid
end

local function obterBasePartDoPrompt(prompt)
	if not prompt or not prompt.Parent then
		return nil
	end

	local pai = prompt.Parent
	if pai:IsA("BasePart") then
		return pai
	elseif pai:IsA("Attachment") and pai.Parent and pai.Parent:IsA("BasePart") then
		return pai.Parent
	elseif pai:IsA("Model") then
		return pai.PrimaryPart or pai:FindFirstChildWhichIsA("BasePart", true)
	end

	return nil
end

local function promptEhDeVenderMinerios(prompt)
	if not prompt or not prompt:IsA("ProximityPrompt") then
		return false
	end

	local acao = string.lower(prompt.ActionText or "")
	local objeto = string.lower(prompt.ObjectText or "")
	local nomePrompt = string.lower(prompt.Name or "")
	local nomePai = prompt.Parent and string.lower(prompt.Parent.Name) or ""
	local nomeAvo = (prompt.Parent and prompt.Parent.Parent) and string.lower(prompt.Parent.Parent.Name) or ""

	return string.find(acao, CONFIG.TextoPromptAlvo, 1, true)
		or string.find(objeto, CONFIG.TextoPromptAlvo, 1, true)
		or string.find(nomePrompt, "vender", 1, true)
		or string.find(nomePai, "vender", 1, true)
		or string.find(nomePai, "miner", 1, true)
		or string.find(nomeAvo, "vender", 1, true)
		or string.find(nomeAvo, "miner", 1, true)
end

local function encontrarPromptNpcVenda()
	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("ProximityPrompt") and promptEhDeVenderMinerios(obj) then
			local parte = obterBasePartDoPrompt(obj)
			if parte and parte:IsDescendantOf(workspace) then
				return obj, parte
			end
		end
	end
	return nil, nil
end

local function modeloPertenceAoJogador(modelo)
	if not modelo or not modelo:IsA("Model") then
		return false
	end

	local nomeJogador = string.lower(player.Name)
	local displayJogador = string.lower(player.DisplayName or "")
	local userIdStr = tostring(player.UserId)

	local function valorEhDoJogador(val)
		if val == nil then
			return false
		end
		if typeof(val) == "Instance" then
			return val == player or val == player.Character
		end
		local s = string.lower(tostring(val))
		return s == nomeJogador
			or (displayJogador ~= "" and s == displayJogador)
			or s == userIdStr
			or string.find(s, nomeJogador, 1, true) ~= nil
			or string.find(s, userIdStr, 1, true) ~= nil
	end

	for chave, valor in pairs(modelo:GetAttributes()) do
		local chaveLower = string.lower(tostring(chave))
		if string.find(chaveLower, "owner", 1, true)
			or string.find(chaveLower, "dono", 1, true)
			or string.find(chaveLower, "player", 1, true)
			or string.find(chaveLower, "user", 1, true)
			or string.find(chaveLower, "driver", 1, true)
			or string.find(chaveLower, "motorista", 1, true)
		then
			if valorEhDoJogador(valor) then
				return true
			end
		end
	end

	for _, desc in ipairs(modelo:GetDescendants()) do
		if desc:IsA("ObjectValue") or desc:IsA("StringValue") or desc:IsA("IntValue") or desc:IsA("NumberValue") then
			local n = string.lower(desc.Name)
			if string.find(n, "owner", 1, true)
				or string.find(n, "dono", 1, true)
				or string.find(n, "player", 1, true)
				or string.find(n, "user", 1, true)
				or string.find(n, "driver", 1, true)
			then
				if valorEhDoJogador(desc.Value) then
					return true
				end
			end
		elseif desc:IsA("Seat") or desc:IsA("VehicleSeat") then
			if desc.Occupant and desc.Occupant.Parent == player.Character then
				return true
			end
		end
	end

	local nomeModelo = string.lower(modelo.Name)
	if string.find(nomeModelo, nomeJogador, 1, true) or string.find(nomeModelo, userIdStr, 1, true) then
		return true
	end

	return false
end

local function encontrarMeuCarroPerto()
	local _, hrp = obterPersonagem()
	if not hrp then
		return nil, nil
	end

	local melhorCarro = nil
	local melhorAssento = nil
	local menorDist = CONFIG.RaioDetectarCarro

	for _, obj in ipairs(workspace:GetDescendants()) do
		if obj:IsA("VehicleSeat") or obj:IsA("Seat") then
			local modelo = obj:FindFirstAncestorOfClass("Model")
			if modelo and modelo ~= player.Character and modeloPertenceAoJogador(modelo) then
				if not obj.Occupant or obj.Occupant.Parent == player.Character then
					local dist = (obj.Position - hrp.Position).Magnitude
					if dist <= menorDist then
						menorDist = dist
						melhorCarro = modelo
						melhorAssento = obj
					end
				end
			end
		end
	end

	return melhorCarro, melhorAssento
end

local function entrarNoCarro(carro, assento)
	local character, hrp, humanoid = obterPersonagem()
	if not character or not hrp or not humanoid or not carro or not assento then
		return false
	end

	if humanoid.SeatPart == assento then
		return true
	end

	character:PivotTo(assento.CFrame * CFrame.new(0, 1.5, 0))
	pcall(function()
		assento:Sit(humanoid)
	end)

	for _ = 1, 15 do
		if humanoid.SeatPart and humanoid.SeatPart:IsDescendantOf(carro) then
			return true
		end
		character:PivotTo(assento.CFrame * CFrame.new(0, 1.2, 0))
		task.wait(0.03)
	end

	return humanoid.SeatPart ~= nil and humanoid.SeatPart:IsDescendantOf(carro)
end

local function sairDoCarro()
	local character, hrp, humanoid = obterPersonagem()
	if not character or not hrp or not humanoid then
		return
	end

	humanoid.Sit = false
	if humanoid.SeatPart then
		pcall(function()
			humanoid.SeatPart.Occupant = nil
		end)
	end
	task.wait(0.03)
	character:PivotTo(hrp.CFrame * CFrame.new(0, 2.5, -3.5))
end

local function teleporteLivre(cfDestino, carro)
	local character, hrp = obterPersonagem()
	if not character or not hrp then
		return false
	end

	if carro then
		local base = carro.PrimaryPart or carro:FindFirstChildWhichIsA("BasePart", true)
		if base then
			local offsetCar = base.CFrame:ToObjectSpace(hrp.CFrame)
			carro:PivotTo(cfDestino)
			character:PivotTo(base.CFrame * offsetCar)
		else
			carro:PivotTo(cfDestino)
			character:PivotTo(cfDestino * CFrame.new(0, 2, 0))
		end
	else
		character:PivotTo(cfDestino)
	end

	return true
end

local function voarDevagarAteNpc(parteNpc)
	local character, hrp = obterPersonagem()
	if not character or not hrp or not parteNpc or not parteNpc.Parent then
		return false
	end

	local inicio = os.clock()
	local maxDuracao = 6.0

	while (os.clock() - inicio) < maxDuracao do
		character, hrp = obterPersonagem()
		if not character or not hrp or not parteNpc.Parent then
			return false
		end

		local alvoPos = parteNpc.Position
		local vetorAteNpc = alvoPos - hrp.Position
		local distancia = vetorAteNpc.Magnitude

		if distancia <= CONFIG.DistanciaParadaNpc then
			hrp.AssemblyLinearVelocity = Vector3.zero
			hrp.AssemblyAngularVelocity = Vector3.zero
			hrp.CFrame = CFrame.lookAt(hrp.Position, Vector3.new(alvoPos.X, hrp.Position.Y, alvoPos.Z))
			return true
		end

		local dt = RunService.Heartbeat:Wait()
		local passo = math.min(distancia - CONFIG.DistanciaParadaNpc * 0.5, CONFIG.VelocidadeAproximacaoNpc * dt)
		if passo > 0 then
			local direcao = vetorAteNpc.Unit
			local novaPos = hrp.Position + (direcao * passo)
			hrp.AssemblyLinearVelocity = Vector3.zero
			hrp.AssemblyAngularVelocity = Vector3.zero
			character:PivotTo(CFrame.lookAt(novaPos, Vector3.new(alvoPos.X, hrp.Position.Y, alvoPos.Z)))
		end
	end

	return true
end

local function segurarAteAbrirMenuVenda(prompt, parteNpc)
    local _, hrp = obterPersonagem()
    if hrp and parteNpc and parteNpc.Parent then
        hrp.AssemblyLinearVelocity = Vector3.zero
        hrp.AssemblyAngularVelocity = Vector3.zero
        hrp.CFrame = CFrame.lookAt(
            hrp.Position,
            Vector3.new(parteNpc.Position.X, hrp.Position.Y, parteNpc.Position.Z)
        )
    end

    -- Aguarda o personagem estabilizar diante do vendedor.
    task.wait(0.5)

    if not prompt or not prompt.Parent or not prompt.Enabled then
        warn("[PRIME X MOBILE] Prompt do vendedor indisponivel.")
        return false
    end

    local holdDuracao = math.max(tonumber(prompt.HoldDuration) or 0, 0.6)
    local tempoSegurar = holdDuracao + 0.7

    -- Uma interação direta, sem procurar textos, nomes ou elementos do menu.
    local iniciou = false
    local ok = pcall(function()
        prompt:InputHoldBegin()
        iniciou = true
    end)

    if iniciou then
        task.wait(tempoSegurar)
        pcall(function()
            prompt:InputHoldEnd()
        end)
        task.wait(1.0)
        return true
    end

    -- Alternativa caso o InputHoldBegin não esteja disponível no executor.
    if typeof(fireproximityprompt) == "function" then
        local disparou = pcall(function()
            fireproximityprompt(prompt, holdDuracao)
        end)
        if disparou then
            task.wait(1.0)
            return true
        end
    end

    warn("[PRIME X MOBILE] Nao foi possivel interagir com o prompt do vendedor.")
    return false
end

-- Toque/clique nas coordenadas da tela mobile.
local function clicarCoordenada(x, y)
    local okDown = pcall(function()
        VirtualInputManager:SendMouseButtonEvent(x, y, 0, true, game, 0)
    end)
    task.wait(0.08)
    local okUp = pcall(function()
        VirtualInputManager:SendMouseButtonEvent(x, y, 0, false, game, 0)
    end)
    if not okDown or not okUp then
        warn("[PRIME X MOBILE] Nao foi possivel tocar na coordenada.")
        return false
    end
    return true
end

local function executarVendaERetorno()
    -- Suspende a camera superior durante todo o trajeto e a venda.
    pararCameraMineracao(true)
	local character, hrp = obterPersonagem()
	if not character or not hrp then
		warn("Personagem ou HumanoidRootPart não encontrado.")
        voltarCameraSuperior()
		return
	end

	local offsetAltura = Vector3.new(0, CONFIG.AlturaLevantadaTp, 0)
	local destinoVendaLevantado = CFrame.new(CONFIG.PosicaoVenda + offsetAltura)
	local destinoRetornoLevantado = CFrame.new(CONFIG.PosicaoRetorno + offsetAltura)

	local carro, assento = encontrarMeuCarroPerto()
	if not carro or not assento then
		warn("Seu carro não foi encontrado no raio de " .. CONFIG.RaioDetectarCarro .. " studs.")
        voltarCameraSuperior()
		return
	end

	if not entrarNoCarro(carro, assento) then
		warn("Não foi possível sentar no seu carro.")
        voltarCameraSuperior()
		return
	end

	teleporteLivre(destinoVendaLevantado, carro)
	task.wait(CONFIG.EsperaCarregarMapa)

	sairDoCarro()
	task.wait(0.1)

	local prompt, parteNpc = encontrarPromptNpcVenda()
	if not prompt or not parteNpc then
		warn("ProximityPrompt de 'Vender Minérios' não encontrado após sair do carro.")
		entrarNoCarro(carro, assento)
		task.wait(0.05)
		teleporteLivre(destinoRetornoLevantado, carro)
		task.wait(CONFIG.EsperaCarregarMapa)
		sairDoCarro()
		task.wait(0.1)
        voltarCameraSuperior()
		return
	end

	voarDevagarAteNpc(parteNpc)
	task.wait(0.05)

	local menuIniciado = segurarAteAbrirMenuVenda(prompt, parteNpc)
    if menuIniciado then
        -- Pequena folga para o painel de venda terminar de abrir no mobile.
        task.wait(0.5)

        local coordenadasVenda = CONFIG.CoordenadasCelular
        local coordenadaFechar = CONFIG.CoordenadaFecharX

        -- Mantem os cinco toques originais, com intervalo entre eles.
        for i = 1, #coordenadasVenda do
            local coord = coordenadasVenda[i]
            clicarCoordenada(coord.x, coord.y)
            task.wait(CONFIG.IntervaloCliqueVender)
        end

        if coordenadaFechar then
            clicarCoordenada(coordenadaFechar.x, coordenadaFechar.y)
            task.wait(0.3)
        end
    else
        warn("[PRIME X MOBILE] Venda ignorada porque a interacao com o vendedor falhou.")
    end

	entrarNoCarro(carro, assento)
	task.wait(0.05)
	teleporteLivre(destinoRetornoLevantado, carro)
	task.wait(CONFIG.EsperaCarregarMapa)
	sairDoCarro()
	task.wait(0.1)
    voltarCameraSuperior()
end

local function miningLoop()
    iniciarCameraSuperior()
    -- Ao ativar, exige que a picareta ja esteja na mao.
    -- Depois de cada venda, pressiona 1; nao procura nem pega ferramenta.
    while State.Running and State.Mining do
        for index, pointCFrame in ipairs(Points) do
            if not State.Running or not State.Mining then break end
            State.PointIndex = index
            State.Status = "Voando para posicao " .. index .. " de " .. #Points
            if not walkTo(pointCFrame) then
                task.wait(0.2)
                break
            end

            -- Fica estabilizado no ponto e espera o prompt de mineracao aparecer.
            local prompt, part
            State.Status = "No ponto " .. index .. "; procurando minerio"
            while State.Running and State.Mining and not prompt do
                prompt = findMiningPromptAtPoint(pointCFrame.Position, 35)
                if prompt then
                    part = getPromptPart(prompt)
                    if not part or not prompt.Enabled then
                        prompt, part = nil, nil
                    end
                end
                if not prompt then
                    holdPosition(pointCFrame.Rotation)
                    State.Status = "Aguardando minerio/prompt aparecer no ponto " .. index
                    task.wait(0.5)
                end
            end
            if not State.Running or not State.Mining then break end

            -- Voa ate uma posicao lateral proxima da pedra; continua ajustando
            -- ate entrar no alcance configurado do ProximityPrompt.
            local _, root = getHumanoidRoot()
            local activationDistance = prompt.MaxActivationDistance or 6
            local away = Vector3.new(root.Position.X - part.Position.X, 0, root.Position.Z - part.Position.Z)
            if away.Magnitude < 0.05 then
                away = Vector3.new(pointCFrame.LookVector.X, 0, pointCFrame.LookVector.Z)
            end
            if away.Magnitude < 0.05 then away = Vector3.new(0, 0, 1) end
            away = away.Unit

            local safeDistance = math.max(1.5, math.min(3.0, activationDistance * 0.55))
            local approachPosition = part.Position + away * safeDistance + Vector3.new(0, 1.5, 0)
            local approachCFrame = CFrame.new(approachPosition) * pointCFrame.Rotation
            State.Status = "Aproximando ate o botao Minerar aparecer"
            if not walkTo(approachCFrame, prompt) then
                State.Status = "Falha ao aproximar do minerio"
                task.wait(0.5)
                break
            end

            -- Espera no local ate o prompt estar ativo e dentro da distancia real.
            local ready = false
            while State.Running and State.Mining and prompt and prompt.Parent do
                _, root = getHumanoidRoot()
                part = getPromptPart(prompt)
                if not part then break end
                local distance = (root.Position - part.Position).Magnitude
                local maxDistance = prompt.MaxActivationDistance or 6
                -- PromptShown confirma que o botao E realmente apareceu na tela.
                -- Assim que aparecer, para o voo na posicao atual e inicia a retencao.
                if prompt.Enabled and State.VisiblePrompts[prompt] then
                    ready = true
                    holdPosition(pointCFrame.Rotation)
                    break
                end

                -- Se ja esta no alcance, fica totalmente parado e espera o
                -- evento visual do prompt. Nao continua se mexendo por cima da pedra.
                if distance <= maxDistance then
                    holdPosition(pointCFrame.Rotation)
                    State.Status = "No alcance; esperando o botao E aparecer"
                    task.wait(0.05)
                else
                    local dir = Vector3.new(root.Position.X - part.Position.X, 0, root.Position.Z - part.Position.Z)
                    if dir.Magnitude < 0.05 then dir = Vector3.new(0, 0, 1) end
                    dir = dir.Unit
                    local nextPos = part.Position + dir * math.max(1.25, maxDistance * 0.45) + Vector3.new(0, 1.5, 0)
                    State.Status = "Aproximando ate o botao E aparecer"
                    walkTo(CFrame.new(nextPos) * pointCFrame.Rotation)
                end
            end

            if ready and State.Running and State.Mining then
                holdPosition(pointCFrame.Rotation)
                State.Status = "Botao E apareceu; segurando por 16 segundos"
                local ok = holdMiningPrompt(prompt)
                if ok then
                    local stones = MineFolder and MineFolder:FindFirstChild("Pedras")
                    local key = getOreKey(prompt, stones)
                    State.LastOre[key] = os.clock()
                    MineriosDesdeVenda += 1
                    State.Status = "Minerio concluido (" .. MineriosDesdeVenda .. "/" .. ORES_POR_VENDA .. "); cooldown de 20s"

                    if MineriosDesdeVenda >= ORES_POR_VENDA and State.Running and State.Mining then
                        State.Status = "9 minerios coletados; iniciando venda"
                        stopFlight()
                        executarVendaERetorno()
                        MineriosDesdeVenda = 0
                        if State.Running and State.Mining then
                            refreshCharacter()
                            State.Status = "Venda concluida; pressionando 1 para equipar a picareta"
                            if not activatePickaxe() then
                                State.Status = "Nao foi possivel pressionar 1 apos a venda"
                                task.wait(0.5)
                            else
                                State.Status = "Tecla 1 pressionada; retomando mineracao"
                            end
                        end
                    end
                else
                    State.Status = "Retencao interrompida antes de completar 16 segundos; minerio nao contado"
                    task.wait(0.5)
                end
            else
                State.Status = "Prompt indisponivel; permanecendo no ponto"
            end

            holdPosition(pointCFrame.Rotation)
            task.wait(0.4)
        end
    end
    stopFlight()
    pararCameraMineracao(true)
    SavedCameraType, SavedCameraSubject = nil, nil
    SavedMinZoom, SavedMaxZoom = nil, nil
    State.Status = "Mineracao desligada"
end



-- Interface do minerador: versao mobile.
local MinerBody = CriarSecao("AUTO MINERADOR", 3, 42, 215)
local MinerStatus = makeLabel(MinerBody, "Status: Aguardando inicio", 2, 34, 11)
local MinerSpeedLabel = makeLabel(MinerBody, "VELOCIDADE DE ANDADA: " .. State.Speed, 37, 22, 11)
local SpeedBox = Instance.new("TextBox")
SpeedBox.Size = UDim2.fromOffset(70, 28)
SpeedBox.Position = UDim2.fromOffset(12, 62)
SpeedBox.BackgroundColor3 = DARK2
SpeedBox.TextColor3 = WHITE
SpeedBox.Font = Enum.Font.GothamBold
SpeedBox.TextSize = 12
SpeedBox.Text = tostring(State.Speed)
SpeedBox.ClearTextOnFocus = false
SpeedBox.Parent = MinerBody
Instance.new("UICorner", SpeedBox).CornerRadius = UDim.new(0, 6)

local MinerStart = makeButton(MinerBody, "▶ ATIVAR AUTO MINERADOR", 12, 120, 0.96, 36, GREEN)
local MinerActivity = makeLabel(MinerBody, "Atividade: Aguardando inicio", 161, 38, 10)

local function atualizarMineradorUI()
    local ativo = State.Mining
    MinerStart.Text = ativo and "■ DESATIVAR AUTO MINERADOR" or "▶ ATIVAR AUTO MINERADOR"
    MinerStart.BackgroundColor3 = ativo and RED or GREEN
    MinerStatus.Text = (ativo and "Status: ATIVO | " or "Status: INATIVO | ") .. tostring(State.Status)
    MinerActivity.Text = "Minerios para venda: " .. tostring(MineriosDesdeVenda) .. "/" .. tostring(ORES_POR_VENDA)
end

MinerStart.MouseButton1Click:Connect(function()
    if not State.Running then return end
    State.Mining = not State.Mining
    if State.Mining then
        -- Na ativacao inicial, so inicia se a picareta ja estiver na mao.
        -- Nao pressiona 1 nem procura/pega a ferramenta nesse momento.
        if not hasEquippedPickaxe() then
            State.Mining = false
            State.Status = "Equipe a picareta na mao antes de ativar"
            atualizarMineradorUI()
            return
        end
        State.Status = "Picareta na mao; iniciando mineracao"
        task.spawn(miningLoop)
    else
        stopFlight()
        pararCameraMineracao(true)
        SavedCameraType, SavedCameraSubject = nil, nil
        SavedMinZoom, SavedMaxZoom = nil, nil
        State.Status = "Mineracao desligada"
    end
    atualizarMineradorUI()
end)

SpeedBox.FocusLost:Connect(function()
    local value = tonumber(SpeedBox.Text)
    if value then State.Speed = math.clamp(value, 8, 100) end
    SpeedBox.Text = tostring(State.Speed)
    MinerSpeedLabel.Text = "VELOCIDADE DE ANDADA: " .. State.Speed
end)

task.spawn(function()
    while Gui.Parent and State.Running do
        atualizarMineradorUI()
        task.wait(0.25)
    end
end)

-- =========================================================
-- AUTO CAIXA
-- =========================================================
local CaixaBody = CriarSecao("AUTO CAIXA", 1, 42, 230)
local CaixaStatus = makeLabel(CaixaBody, "Status: DESATIVADO", 2, 22, 12)
local CaixaProgresso = makeLabel(CaixaBody, "Aguardando...", 25, 22, 11)
local CaixaSpeedLabel = makeLabel(CaixaBody, "Speed do Caixa: 24  |  Limite: 8-100", 49, 22, 11)
local CaixaMenos = makeButton(CaixaBody, "− SPEED CAIXA", 12, 77, 0.47, 34, Color3.fromRGB(72,48,103))
local CaixaMais = makeButton(CaixaBody, "+ SPEED CAIXA", 0, 77, 0.47, 34, Color3.fromRGB(105,62,160))
CaixaMais.AnchorPoint = Vector2.new(1, 0)
CaixaMais.Position = UDim2.new(1, -12, 0, 77)
local CaixaStart = makeButton(CaixaBody, "▶ ATIVAR AUTO CAIXA", 12, 120, 0.96, 36, OFF)

local Caixa, Entrega, CaixaPrompt, EntregaPrompt
local function AtualizarReferenciasCaixa()
    local construcoes = Workspace:FindFirstChild("Construcoes")
    local prisao = construcoes and construcoes:FindFirstChild("PrisaoFolder")
    local emprego = prisao and prisao:FindFirstChild("EmpregoPreso")
    Caixa = emprego and emprego:FindFirstChild("LocalCaixa")
    Entrega = emprego and emprego:FindFirstChild("LocalEntrega")
    CaixaPrompt = Caixa and Caixa:FindFirstChildWhichIsA("ProximityPrompt", true)
    EntregaPrompt = Entrega and Entrega:FindFirstChildWhichIsA("ProximityPrompt", true)
    return Caixa and Entrega and CaixaPrompt and EntregaPrompt
end
local function StatusCaixa(t) CaixaStatus.Text = "Status: " .. t end
local function Posicao(obj)
    if not obj then return nil end
    if obj:IsA("Attachment") then return obj.WorldPosition end
    if obj:IsA("BasePart") then return obj.Position end
    if obj:IsA("Model") then return obj:GetPivot().Position end
    local part = obj:FindFirstAncestorWhichIsA("BasePart")
    return part and part.Position
end
local function Aproximar(obj, prompt)
    local root = Personagem()
    local target = Posicao(obj)
    if not root or not target then return nil end
    local direction = root.Position - target
    if direction.Magnitude <= prompt.MaxActivationDistance * 0.55 then return root.Position end
    if direction.Magnitude < 0.1 then direction = Vector3.new(1,0,0) end
    local distance = math.max(1, math.min(prompt.MaxActivationDistance * 0.55, 5))
    return target + direction.Unit * distance
end
local function MoverCaixa(position)
    local root, humanoid = Personagem()
    if not root or not humanoid or not Hub.CaixaAtivo then return false end
    humanoid.WalkSpeed = Hub.CaixaVelocidade
    humanoid:MoveTo(position)
    local terminou, chegou = false, false
    local connection = humanoid.MoveToFinished:Connect(function(result) chegou = result; terminou = true end)
    local timeout = os.clock() + 20
    while Hub.CaixaAtivo and not terminou and os.clock() < timeout do
        root, humanoid = Personagem()
        if not root or not humanoid then connection:Disconnect(); return false end
        humanoid.WalkSpeed = Hub.CaixaVelocidade
        if (root.Position - position).Magnitude <= 3 then chegou = true; terminou = true; break end
        task.wait(0.1)
    end
    connection:Disconnect()
    return Hub.CaixaAtivo and chegou
end
local function Interagir(prompt)
    if not Hub.CaixaAtivo or not prompt.Parent or not prompt.Enabled then return false end
    local root = Personagem()
    local target = Posicao(prompt.Parent)
    if not root or not target or (root.Position - target).Magnitude > prompt.MaxActivationDistance + 1 then return false end
    CaixaProgresso.Text = prompt.ActionText
    if typeof(fireproximityprompt) == "function" then
        local ok = pcall(function() fireproximityprompt(prompt, prompt.HoldDuration) end)
        if ok then task.wait(prompt.HoldDuration + 0.5); return Hub.CaixaAtivo end
    end
    local ok = pcall(function()
        prompt:InputHoldBegin()
        task.wait(prompt.HoldDuration + 0.3)
        prompt:InputHoldEnd()
    end)
    return ok and Hub.CaixaAtivo
end
local function EtapaCaixa(obj, prompt, nome)
    if not Hub.CaixaAtivo then return false end
    StatusCaixa("INDO PARA " .. nome)
    CaixaProgresso.Text = "Calculando caminho..."
    local destino = Aproximar(obj, prompt)
    if not destino or not MoverCaixa(destino) then
        CaixaProgresso.Text = "Falha no caminho"
        return false
    end
    if not Hub.CaixaAtivo then return false end
    task.wait(0.3)
    local root = Personagem()
    local target = Posicao(prompt.Parent)
    if not root or not target or (root.Position - target).Magnitude > prompt.MaxActivationDistance then
        CaixaProgresso.Text = "Fora do alcance"
        return false
    end
    StatusCaixa("INTERAGINDO")
    return Interagir(prompt)
end
local function ExecutarCaixa()
    while Hub.CaixaAtivo do
        if not AtualizarReferenciasCaixa() then
            StatusCaixa("AGUARDANDO AREA DA PRISAO")
            CaixaProgresso.Text = "Entre na area da prisao para iniciar..."
            task.wait(1)
            continue
        end
        if not AdquirirMovimento("CAIXA", function() return Hub.CaixaAtivo end) then break end
        if not AtualizarReferenciasCaixa() then
            LiberarMovimento("CAIXA")
            task.wait(0.5)
            continue
        end
        local sucesso = false
        local ok = pcall(function()
            StatusCaixa("INDO PEGAR CAIXA")
            if not EtapaCaixa(Caixa, CaixaPrompt, "CAIXA") or not Hub.CaixaAtivo then return end
            StatusCaixa("INDO ENTREGAR")
            if EtapaCaixa(Entrega, EntregaPrompt, "ENTREGA") and Hub.CaixaAtivo then
                CaixaProgresso.Text = "ENTREGA CONCLUIDA!"
                sucesso = true
            end
        end)
        LiberarMovimento("CAIXA")
        if not ok or not sucesso then break end
        task.wait(1)
    end
    Hub.CaixaAtivo = false
    LiberarMovimento("CAIXA")
    StatusCaixa("DESATIVADO")
    setButtonActive(CaixaStart, false, "■ DESATIVAR AUTO CAIXA", "▶ ATIVAR AUTO CAIXA")
end
CaixaMenos.MouseButton1Click:Connect(function()
    Hub.CaixaVelocidade = math.max(8, Hub.CaixaVelocidade - 4)
    CaixaSpeedLabel.Text = "Speed do Caixa: " .. Hub.CaixaVelocidade .. "  |  Limite: 8-100"
end)
CaixaMais.MouseButton1Click:Connect(function()
    Hub.CaixaVelocidade = math.min(100, Hub.CaixaVelocidade + 4)
    CaixaSpeedLabel.Text = "Speed do Caixa: " .. Hub.CaixaVelocidade .. "  |  Limite: 8-100"
end)
CaixaStart.MouseButton1Click:Connect(function()
    Hub.CaixaAtivo = not Hub.CaixaAtivo
    if Hub.CaixaAtivo then
        setButtonActive(CaixaStart, true, "■ DESATIVAR AUTO CAIXA", "▶ ATIVAR AUTO CAIXA")
        StatusCaixa("ATIVADO")
        task.spawn(ExecutarCaixa)
    else
        StatusCaixa("PARANDO...")
        CaixaProgresso.Text = "Interrompendo..."
    end
end)

-- =========================================================
-- AUTO EMPILHADOR V9
-- =========================================================
local EmpBody = CriarSecao("AUTO EMPILHADOR V9", 2, 42, 185)
local EmpStatus = makeLabel(EmpBody, "Entre na empilhadeira.", 3, 35, 11)
local EmpInfo = makeLabel(EmpBody, "Speed: 65  |  Altura: 20  |  Espera: 1s", 39, 22, 10)
local EmpButton = makeButton(EmpBody, "▶ ATIVAR EMPILHADOR", 12, 75, 0.96, 38, GREEN)

local SPEED = 65
local ALTURA_VOO = 20
local TEMPO_ENTREGA = 1

local COLETA_ORIGINAL = CFrame.new(
    2985.808838, 4.134086, 3148.478760,
    0.992718, 0, 0.120459,
    0, 1, 0,
    -0.120459, 0, 0.992718
)
local ENTREGA_ORIGINAL = CFrame.new(
    2936.614746, 22.068596, 3250.278564,
    -0.073422, 0, -0.997301,
    0, 1, 0,
    0.997301, 0, -0.073422
)
local function inverter(cf) return cf * CFrame.Angles(0, math.pi, 0) end
local COLETA = inverter(COLETA_ORIGINAL)
local ENTREGA = inverter(ENTREGA_ORIGINAL)

local function atualizarEmpStatus(t) EmpStatus.Text = t end
local function atualizarEmpBotao()
    EmpButton.Text = Emp.Running and "■ DESATIVAR EMPILHADOR" or "▶ ATIVAR EMPILHADOR"
    EmpButton.BackgroundColor3 = Emp.Running and RED or GREEN
end

local function encontrarEmpilhadeira()
    local pasta = Workspace:FindFirstChild("CarrosSpawnados")
    local carro = pasta and pasta:FindFirstChild("Empilhadeira")
    if not carro then return nil, "Empilhadeira nao encontrada." end
    local personagem = Player.Character
    local humanoid = personagem and personagem:FindFirstChildOfClass("Humanoid")
    if not humanoid or not humanoid.SeatPart then return nil, "Entre na empilhadeira primeiro." end
    local seat = carro:FindFirstChild("PPDriveSeat", true)
    local body = carro:FindFirstChild("Body")
    local frente = body and body:FindFirstChild("Mec")
    if not seat or not seat:IsA("BasePart") then return nil, "Banco nao encontrado." end
    if not frente or not frente:IsA("BasePart") then return nil, "Body.Mec nao encontrada." end
    if not humanoid.SeatPart:IsDescendantOf(carro) then return nil, "Voce nao esta dirigindo." end
    local direcao = seat.CFrame.LookVector
    local metade = math.abs(direcao:Dot(frente.CFrame.RightVector)) * frente.Size.X / 2
        + math.abs(direcao:Dot(frente.CFrame.UpVector)) * frente.Size.Y / 2
        + math.abs(direcao:Dot(frente.CFrame.LookVector)) * frente.Size.Z / 2
    local pontoFrontal = frente.Position + direcao * metade
    local referencia = CFrame.new(pontoFrontal) * seat.CFrame.Rotation
    local offset = carro:GetPivot():ToObjectSpace(referencia)
    return {vehicle=carro, offset=offset}
end
local function obterFrente(ctx) return ctx.vehicle:GetPivot() * ctx.offset end
local function continuaSentado(ctx)
    local personagem = Player.Character
    local humanoid = personagem and personagem:FindFirstChildOfClass("Humanoid")
    return humanoid and humanoid.SeatPart and humanoid.SeatPart:IsDescendantOf(ctx.vehicle)
end
local function empAtivo(token) return Emp.Running and Emp.Token == token end
local function moverPara(ctx, destino, token)
    local origem = obterFrente(ctx)
    local distancia = (destino.Position - origem.Position).Magnitude
    local duracao = math.max(distancia / SPEED, 0.12)
    local inicio = os.clock()
    while true do
        if not empAtivo(token) then return false end
        if not continuaSentado(ctx) then error("Voce saiu da empilhadeira.") end
        local alpha = math.clamp((os.clock() - inicio) / duracao, 0, 1)
        local novaFrente = origem:Lerp(destino, alpha)
        local novoPivot = novaFrente * ctx.offset:Inverse()
        ctx.vehicle:PivotTo(novoPivot)
        if alpha >= 1 then break end
        RunService.Heartbeat:Wait()
    end
    return empAtivo(token)
end
local function voarAte(ctx, destino, token)
    local atual = obterFrente(ctx)
    local altura = math.max(atual.Position.Y, destino.Position.Y) + ALTURA_VOO
    local subir = CFrame.new(atual.Position.X, altura, atual.Position.Z) * destino.Rotation
    if not moverPara(ctx, subir, token) then return false end
    local atravessar = CFrame.new(destino.Position.X, altura, destino.Position.Z) * destino.Rotation
    if not moverPara(ctx, atravessar, token) then return false end
    return moverPara(ctx, destino, token)
end
local function esperarEmp(segundos, token)
    local inicio = os.clock()
    while os.clock() - inicio < segundos do
        if not empAtivo(token) then return false end
        task.wait(0.05)
    end
    return empAtivo(token)
end
local function iniciarEmpilhador()
    local ctx, erro = encontrarEmpilhadeira()
    if not ctx then
        Emp.Running = false
        atualizarEmpBotao()
        atualizarEmpStatus(erro)
        return
    end
    local token = Emp.Token
    task.spawn(function()
        local sucesso, mensagem = pcall(function()
            while empAtivo(token) do
                -- Compartilha a trava com Auto Caixa.
                if not AdquirirMovimento("EMPILHADOR", function() return empAtivo(token) end) then break end
                if not continuaSentado(ctx) then
                    LiberarMovimento("EMPILHADOR")
                    error("Voce saiu da empilhadeira.")
                end
                atualizarEmpStatus("Indo para a madeira...")
                local okColeta = voarAte(ctx, COLETA, token)
                if not okColeta then LiberarMovimento("EMPILHADOR"); break end
                atualizarEmpStatus("Indo para entrega...")
                local okEntrega = voarAte(ctx, ENTREGA, token)
                LiberarMovimento("EMPILHADOR")
                if not okEntrega then break end
                atualizarEmpStatus("Entrega concluida! Esperando 1s...")
                if not esperarEmp(TEMPO_ENTREGA, token) then break end
            end
        end)
        LiberarMovimento("EMPILHADOR")
        if Emp.Token == token then
            Emp.Running = false
            atualizarEmpBotao()
            if sucesso then atualizarEmpStatus("Ciclo desativado.")
            else atualizarEmpStatus("Parado: " .. tostring(mensagem)) end
        end
    end)
end
EmpButton.MouseButton1Click:Connect(function()
    if Emp.Running then
        Emp.Running = false
        Emp.Token = Emp.Token + 1
        atualizarEmpBotao()
        atualizarEmpStatus("Desativando...")
    else
        Emp.Running = true
        Emp.Token = Emp.Token + 1
        atualizarEmpBotao()
        atualizarEmpStatus("Iniciando ciclo...")
        iniciarEmpilhador()
    end
end)

-- Efeito visual da borda
TweenService:Create(Stroke, TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true), {Transparency=0.55}):Play()

atualizarEscala(escalaInicial)
print("PRIME X MOBILE carregado!")
