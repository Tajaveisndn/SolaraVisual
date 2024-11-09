--[[ 
    SolaraHub UI Library
    Versão: 3.0.0
    Autor: Seu Nome
]]
    
local Library = {
    Flags = {},
    Theme = {
        Main = Color3.fromRGB(30, 30, 30),
        Secondary = Color3.fromRGB(45, 45, 45),
        Stroke = Color3.fromRGB(60, 60, 60),
        Accent = Color3.fromRGB(44, 120, 224),
        Text = Color3.fromRGB(255, 255, 255),
        TextDark = Color3.fromRGB(200, 200, 200),
        Dropdown = {
            Background = Color3.fromRGB(50, 50, 50),
            Hover = Color3.fromRGB(70, 70, 70),
            Selected = Color3.fromRGB(60, 60, 60),
            Arrow = Color3.fromRGB(200, 200, 200)
        },
        Tab = {
            Background = Color3.fromRGB(50, 50, 50),
            Hover = Color3.fromRGB(70, 70, 70),
            Selected = Color3.fromRGB(60, 60, 60)
        }
    }
}

-- Serviços
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

-- Variáveis
local Connections = {}
local minimized = false
local configPath = "SolaraHubConfig.json"

-- Funções Utilitárias
local function Create(instance, properties, children)
    local object = Instance.new(instance)
    
    for i, v in pairs(properties or {}) do
        object[i] = v
    end
    
    for i, v in pairs(children or {}) do
        v.Parent = object
    end
    
    return object
end

local function MakeDraggable(topbar, frame)
    local dragging = false
    local dragInput, mousePos, framePos

    topbar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            mousePos = input.Position
            framePos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    topbar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - mousePos
            frame.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
        end
    end)
end

local function Tween(instance, properties, duration, easingStyle, easingDirection)
    TweenService:Create(instance, TweenInfo.new(duration, easingStyle or Enum.EasingStyle.Quad, easingDirection or Enum.EasingDirection.Out), properties):Play()
end

local function LoadConfig()
    local success, data = pcall(function()
        return HttpService:GetAsync(configPath)
    end)
    if success and data then
        local decoded = HttpService:JSONDecode(data)
        return decoded
    else
        return {}
    end
end

local function SaveConfig(config)
    local encoded = HttpService:JSONEncode(config)
    pcall(function()
        HttpService:PostAsync(configPath, encoded, Enum.HttpContentType.ApplicationJson)
    end)
end

-- Função para criar Dropdown com animações avançadas
local function CreateDropdown(parent, options, default, callback)
    local dropdown = Create("Frame", {
        Parent = parent,
        BackgroundColor3 = Library.Theme.Dropdown.Background,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 30)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 4),
        Parent = dropdown
    })

    local dropdownButton = Create("TextButton", {
        Parent = dropdown,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 10, 0, 5),
        Size = UDim2.new(1, -40, 1, -10),
        Font = Enum.Font.Gotham,
        Text = default or "Selecionar",
        TextColor3 = Library.Theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left,
        AutoButtonColor = false
    })

    local arrow = Create("ImageLabel", {
        Parent = dropdown,
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 16, 0, 16),
        Position = UDim2.new(1, -26, 0.5, -8),
        Image = "rbxassetid://6034818376", -- ícone de seta para baixo
        ImageColor3 = Library.Theme.Dropdown.Arrow,
        Rotation = 0
    })

    local dropdownList = Create("Frame", {
        Parent = dropdown,
        BackgroundColor3 = Library.Theme.Dropdown.Background,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 0, 1, 0),
        Size = UDim2.new(1, 0, 0, 0),
        ClipsDescendants = true,
        Visible = false
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 4),
        Parent = dropdownList
    })

    local dropdownLayout = Create("UIListLayout", {
        Parent = dropdownList,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 2)
    })

    -- Animação para abrir/fechar o dropdown
    local function ToggleDropdown()
        if dropdownList.Visible then
            Tween(dropdownList, {Size = UDim2.new(1, 0, 0, 0)}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            Tween(arrow, {Rotation = 0}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            wait(0.3)
            dropdownList.Visible = false
        else
            dropdownList.Visible = true
            Tween(dropdownList, {Size = UDim2.new(1, 0, 0, #options * 25)}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
            Tween(arrow, {Rotation = 180}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
        end
    end

    dropdownButton.MouseButton1Click:Connect(ToggleDropdown)
    arrow.MouseButton1Click:Connect(ToggleDropdown)

    for _, option in ipairs(options) do
        local optionButton = Create("TextButton", {
            Parent = dropdownList,
            BackgroundColor3 = Library.Theme.Dropdown.Background,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 0, 25),
            Font = Enum.Font.Gotham,
            Text = option,
            TextColor3 = Library.Theme.Text,
            TextSize = 14,
            TextXAlignment = Enum.TextXAlignment.Left,
            AutoButtonColor = false
        })

        Create("UICorner", {
            CornerRadius = UDim.new(0, 4),
            Parent = optionButton
        })

        optionButton.MouseEnter:Connect(function()
            Tween(optionButton, {BackgroundColor3 = Library.Theme.Dropdown.Hover}, 0.2)
        end)

        optionButton.MouseLeave:Connect(function()
            Tween(optionButton, {BackgroundColor3 = Library.Theme.Dropdown.Background}, 0.2)
        end)

        optionButton.MouseButton1Click:Connect(function()
            dropdownButton.Text = option
            ToggleDropdown()
            if callback then
                callback(option)
            end
        end)
    end

    return dropdown
end

-- Função para criar animações de fade
local function FadeIn(instance, duration)
    instance.BackgroundTransparency = 1
    instance:TweenSizeAndPosition(instance.Size, instance.Position, "Out", "Quad", duration, true)
    Tween(instance, {BackgroundTransparency = 0}, duration, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
end

-- Função para criar animações de fade out
local function FadeOut(instance, duration)
    Tween(instance, {BackgroundTransparency = 1}, duration, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    instance:TweenSizeAndPosition(instance.Size, instance.Position, "In", "Quad", duration, true)
    wait(duration)
    instance.Visible = false
end

-- Função principal para criar a janela
function Library:Window(title, keybind)
    local Window = {}
    local config = LoadConfig()

    -- Criar GUI Principal
    local SolaraHub = Create("ScreenGui", {
        Name = "SolaraHub",
        Parent = CoreGui,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
        IgnoreGuiInset = true
    })

    -- Frame Principal
    local Main = Create("Frame", {
        Name = "Main",
        Parent = SolaraHub,
        BackgroundColor3 = Library.Theme.Main,
        BorderSizePixel = 0,
        Position = UDim2.new(0.5, -300, 0.5, -175),
        Size = UDim2.new(0, 600, 0, 350),
        ClipsDescendants = true
    })

    -- Adicionar UICorner e Sombra
    Create("UICorner", {
        CornerRadius = UDim.new(0, 8),
        Parent = Main
    })

    local Shadow = Create("Frame", {
        Parent = Main,
        BackgroundColor3 = Color3.new(0, 0, 0),
        BorderSizePixel = 0,
        Position = UDim2.new(0, -5, 0, -5),
        Size = UDim2.new(1, 10, 1, 10),
        ZIndex = Main.ZIndex - 1,
        BackgroundTransparency = 0.5
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 8),
        Parent = Shadow
    })

    -- Adicionar Gradiente ao Frame Principal
    Create("UIGradient", {
        Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0.00, Library.Theme.Main),
            ColorSequenceKeypoint.new(1.00, Library.Theme.Secondary)
        },
        Parent = Main
    })

    -- Topbar
    local Topbar = Create("Frame", {
        Name = "Topbar",
        Parent = Main,
        BackgroundColor3 = Library.Theme.Secondary,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 40)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 8),
        Parent = Topbar
    })

    -- Título
    local Title = Create("TextLabel", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 15, 0, 0),
        Size = UDim2.new(0.7, -15, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = title,
        TextColor3 = Library.Theme.Text,
        TextSize = 18,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    -- Botão de Fechar
    local CloseButton = Create("TextButton", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -40, 0, 0),
        Size = UDim2.new(0, 40, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = "×",
        TextColor3 = Library.Theme.Text,
        TextSize = 24,
        AutoButtonColor = false
    })

    -- Efeitos de Hover no Botão de Fechar
    CloseButton.MouseEnter:Connect(function()
        Tween(CloseButton, {TextColor3 = Color3.fromRGB(255, 0, 0)}, 0.2)
    end)

    CloseButton.MouseLeave:Connect(function()
        Tween(CloseButton, {TextColor3 = Library.Theme.Text}, 0.2)
    end)

    -- Função de Fechar a Janela com Animação
    CloseButton.MouseButton1Click:Connect(function()
        FadeOut(Main, 0.3)
        wait(0.3)
        SolaraHub:Destroy()
    end)

    -- Tornar a Janela Arrastável
    MakeDraggable(Topbar, Main)

    -- Container para as Tabs
    local TabHolder = Create("Frame", {
        Name = "TabHolder",
        Parent = Main,
        BackgroundColor3 = Library.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 10, 0, 50),
        Size = UDim2.new(0, 150, 1, -60)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 8),
        Parent = TabHolder
    })

    -- Lista de Tabs com Scrolling
    local TabList = Create("ScrollingFrame", {
        Name = "TabList",
        Parent = TabHolder,
        Active = true,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 0, 0, 5),
        Size = UDim2.new(1, 0, 1, -10),
        CanvasSize = UDim2.new(0, 0, 0, 0),
        ScrollBarThickness = 4,
        ScrollBarImageColor3 = Library.Theme.Accent,
        ScrollBarImageTransparency = 0.5
    })

    Create("UIListLayout", {
        Parent = TabList,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 5)
    })

    -- Container para o Conteúdo das Tabs
    local Container = Create("Frame", {
        Name = "Container",
        Parent = Main,
        BackgroundColor3 = Library.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 170, 0, 50),
        Size = UDim2.new(1, -180, 1, -60)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 8),
        Parent = Container
    })

    Create("UIListLayout", {
        Parent = Container,
        SortOrder = Enum.SortOrder.LayoutOrder
    })

    -- Método para criar Tabs
    function Window:Tab(name)
        local Tab = {}
        local TabButton
        local TabContent

        -- Criar Botão da Tab
        TabButton = Create("TextButton", {
            Parent = TabList,
            BackgroundColor3 = Library.Theme.Tab.Background,
            BorderSizePixel = 0,
            Size = UDim2.new(1, -10, 0, 30),
            Font = Enum.Font.GothamBold,
            Text = name,
            TextColor3 = Library.Theme.TextDark,
            TextSize = 14,
            TextXAlignment = Enum.TextXAlignment.Left,
            AutoButtonColor = false
        })

        Create("UICorner", {
            CornerRadius = UDim.new(0, 4),
            Parent = TabButton
        })

        -- Efeitos de Hover nas Tabs
        TabButton.MouseEnter:Connect(function()
            if not TabButton.Selected then
                Tween(TabButton, {BackgroundColor3 = Library.Theme.Tab.Hover}, 0.2)
            end
        end)

        TabButton.MouseLeave:Connect(function()
            if not TabButton.Selected then
                Tween(TabButton, {BackgroundColor3 = Library.Theme.Tab.Background}, 0.2)
            end
        end)

        -- Criar Conteúdo da Tab
        TabContent = Create("Frame", {
            Parent = Container,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 1, 0),
            Visible = false
        })

        -- Função para Selecionar a Tab
        local function Select()
            for _, child in ipairs(TabList:GetChildren()) do
                if child:IsA("TextButton") then
                    child.Selected = false
                    child.TextColor3 = Library.Theme.TextDark
                    Tween(child, {BackgroundColor3 = Library.Theme.Tab.Background}, 0.2)
                end
            end
            TabButton.Selected = true
            TabButton.TextColor3 = Library.Theme.Text
            Tween(TabButton, {BackgroundColor3 = Library.Theme.Tab.Selected}, 0.2)
            for _, content in ipairs(Container:GetChildren()) do
                if content:IsA("Frame") then
                    content.Visible = false
                end
            end
            TabContent.Visible = true
        end

        TabButton.MouseButton1Click:Connect(Select)

        -- Selecionar a primeira tab automaticamente
        if #TabList:GetChildren() == 1 then
            Select()
        end

        -- Métodos para adicionar componentes na Tab
        function Tab:Label(text)
            local Label = Create("TextLabel", {
                Parent = TabContent,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, -20, 0, 20),
                Position = UDim2.new(0, 10, 0, 10 + (#TabContent:GetChildren() * 30)),
                Font = Enum.Font.Gotham,
                Text = text,
                TextColor3 = Library.Theme.Text,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })
            return Label
        end

        function Tab:Button(text, callback)
            local Button = Create("TextButton", {
                Parent = TabContent,
                BackgroundColor3 = Library.Theme.Accent,
                BorderSizePixel = 0,
                Size = UDim2.new(1, -20, 0, 30),
                Position = UDim2.new(0, 10, 0, 10 + (#TabContent:GetChildren() * 35)),
                Font = Enum.Font.GothamBold,
                Text = text,
                TextColor3 = Library.Theme.Text,
                TextSize = 14,
                AutoButtonColor = false
            })

            Create("UICorner", {
                CornerRadius = UDim.new(0, 4),
                Parent = Button
            })

            Button.MouseEnter:Connect(function()
                Tween(Button, {BackgroundColor3 = Library.Theme.TextDark}, 0.2)
            end)

            Button.MouseLeave:Connect(function()
                Tween(Button, {BackgroundColor3 = Library.Theme.Accent}, 0.2)
            end)

            Button.MouseButton1Click:Connect(function()
                if callback then
                    callback()
                end
            end)

            return Button
        end

        function Tab:Dropdown(options, default, callback)
            return CreateDropdown(TabContent, options, default, callback)
        end

        function Tab:Toggle(text, default, callback)
            local Toggle = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Library.Theme.Dropdown.Background,
                BorderSizePixel = 0,
                Size = UDim2.new(1, -20, 0, 30),
                Position = UDim2.new(0, 10, 0, 10 + (#TabContent:GetChildren() * 35))
            })

            Create("UICorner", {
                CornerRadius = UDim.new(0, 4),
                Parent = Toggle
            })

            local ToggleLabel = Create("TextLabel", {
                Parent = Toggle,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 5),
                Size = UDim2.new(0.7, -10, 1, -10),
                Font = Enum.Font.Gotham,
                Text = text,
                TextColor3 = Library.Theme.Text,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local ToggleButton = Create("TextButton", {
                Parent = Toggle,
                BackgroundColor3 = default and Library.Theme.Accent or Library.Theme.TextDark,
                BorderSizePixel = 0,
                Size = UDim2.new(0, 20, 0, 20),
                Position = UDim2.new(1, -30, 0.5, -10),
                Text = "",
                AutoButtonColor = false
            })

            Create("UICorner", {
                CornerRadius = UDim.new(1, 0),
                Parent = ToggleButton
            })

            local toggled = default or false

            ToggleButton.MouseButton1Click:Connect(function()
                toggled = not toggled
                Tween(ToggleButton, {BackgroundColor3 = toggled and Library.Theme.Accent or Library.Theme.TextDark}, 0.2)
                if callback then
                    callback(toggled)
                end
            end)

            return ToggleButton
        end

        return Tab
    end

    -- Configuração do Keybind para Minimizar
    if keybind then
        UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if gameProcessed then return end
            if input.KeyCode == keybind then
                minimized = not minimized
                if minimized then
                    Tween(Main, {Size = UDim2.new(0, 600, 0, 40)}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
                else
                    Tween(Main, {Size = UDim2.new(0, 600, 0, 350)}, 0.3, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
                end
            end
        end)
    end

    -- Salvar Configurações quando a Janela for Fechada
    SolaraHub:BindToClose(function()
        SaveConfig(Library.Flags)
    end)

    -- Método para Fechar a Janela
    function Window:Close()
        FadeOut(Main, 0.3)
        wait(0.3)
        SolaraHub:Destroy()
    end

    return Window
end

return Library
