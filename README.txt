-- Solara Library
-- Created by Claude
-- Version 1.0

local Solara = {}
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

-- Configurações principais
local config = {
    AnimationDuration = 0.3,
    EasingStyle = Enum.EasingStyle.Quint,
    EasingDirection = Enum.EasingDirection.Out,
    MainColor = Color3.fromRGB(32, 32, 32),
    AccentColor = Color3.fromRGB(0, 120, 255),
    TextColor = Color3.fromRGB(255, 255, 255),
    Font = Enum.Font.GothamMedium
}

function Solara.new(title)
    local GUI = Instance.new("ScreenGui")
    local Main = Instance.new("Frame")
    local TopBar = Instance.new("Frame")
    local Title = Instance.new("TextLabel")
    local Container = Instance.new("Frame")
    local TabContainer = Instance.new("Frame")
    local UIListLayout = Instance.new("UIListLayout")
    
    -- Configuração do GUI principal
    GUI.Name = "SolaraLibrary"
    GUI.Parent = game.CoreGui
    
    Main.Name = "Main"
    Main.Parent = GUI
    Main.BackgroundColor3 = config.MainColor
    Main.Position = UDim2.new(0.5, -250, 0.5, -150)
    Main.Size = UDim2.new(0, 500, 0, 300)
    Main.ClipsDescendants = true
    
    -- Barra superior
    TopBar.Name = "TopBar"
    TopBar.Parent = Main
    TopBar.BackgroundColor3 = config.AccentColor
    TopBar.Size = UDim2.new(1, 0, 0, 30)
    
    -- Logo
    local Logo = Instance.new("ImageLabel")
    Logo.Parent = TopBar
    Logo.BackgroundTransparency = 1
    Logo.Position = UDim2.new(0, 5, 0, 5)
    Logo.Size = UDim2.new(0, 20, 0, 20)
    Logo.Image = "rbxassetid://134529686872246" -- ID da logo Solara
    
    Title.Name = "Title"
    Title.Parent = TopBar
    Title.BackgroundTransparency = 1
    Title.Position = UDim2.new(0, 30, 0, 0)
    Title.Size = UDim2.new(1, -30, 1, 0)
    Title.Font = config.Font
    Title.Text = title
    Title.TextColor3 = config.TextColor
    Title.TextSize = 14
    
    -- Container principal
    Container.Name = "Container"
    Container.Parent = Main
    Container.BackgroundTransparency = 1
    Container.Position = UDim2.new(0, 0, 0, 30)
    Container.Size = UDim2.new(1, 0, 1, -30)
    
    -- Sistema de Tabs
    TabContainer.Name = "TabContainer"
    TabContainer.Parent = Container
    TabContainer.BackgroundTransparency = 1
    TabContainer.Position = UDim2.new(0, 0, 0, 0)
    TabContainer.Size = UDim2.new(0.2, 0, 1, 0)
    
    UIListLayout.Parent = TabContainer
    UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder
    UIListLayout.Padding = UDim.new(0, 5)
    
    -- Funções de animação
    local function CreateTween(instance, properties)
        local tween = TweenService:Create(
            instance,
            TweenInfo.new(config.AnimationDuration, config.EasingStyle, config.EasingDirection),
            properties
        )
        return tween
    end
    
    -- Sistema de arraste
    local dragging
    local dragInput
    local dragStart
    local startPos
    
    TopBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = Main.Position
        end
    end)
    
    TopBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and dragging then
            local delta = input.Position - dragStart
            Main.Position = UDim2.new(
                startPos.X.Scale,
                startPos.X.Offset + delta.X,
                startPos.Y.Scale,
                startPos.Y.Offset + delta.Y
            )
        end
    end)
    
    -- Métodos da biblioteca
    local library = {}
    
    function library:CreateTab(name)
        local Tab = Instance.new("TextButton")
        local TabContent = Instance.new("Frame")
        local ElementList = Instance.new("UIListLayout")
        
        Tab.Name = name
        Tab.Parent = TabContainer
        Tab.BackgroundColor3 = config.MainColor
        Tab.Size = UDim2.new(1, -10, 0, 30)
        Tab.Font = config.Font
        Tab.Text = name
        Tab.TextColor3 = config.TextColor
        Tab.TextSize = 14
        
        TabContent.Name = name.."Content"
        TabContent.Parent = Container
        TabContent.BackgroundTransparency = 1
        TabContent.Position = UDim2.new(0.2, 0, 0, 0)
        TabContent.Size = UDim2.new(0.8, 0, 1, 0)
        TabContent.Visible = false
        
        ElementList.Parent = TabContent
        ElementList.SortOrder = Enum.SortOrder.LayoutOrder
        ElementList.Padding = UDim.new(0, 5)
        
        local tabFunctions = {}
        
        -- Função para criar Toggle
        function tabFunctions:CreateToggle(text, callback)
            local Toggle = Instance.new("Frame")
            local Title = Instance.new("TextLabel")
            local Button = Instance.new("TextButton")
            local enabled = false
            
            Toggle.Name = "Toggle"
            Toggle.Parent = TabContent
            Toggle.BackgroundTransparency = 1
            Toggle.Size = UDim2.new(1, -20, 0, 30)
            
            Title.Name = "Title"
            Title.Parent = Toggle
            Title.BackgroundTransparency = 1
            Title.Size = UDim2.new(0.7, 0, 1, 0)
            Title.Font = config.Font
            Title.Text = text
            Title.TextColor3 = config.TextColor
            Title.TextSize = 14
            Title.TextXAlignment = Enum.TextXAlignment.Left
            
            Button.Name = "Button"
            Button.Parent = Toggle
            Button.BackgroundColor3 = config.MainColor
            Button.Position = UDim2.new(0.8, 0, 0.15, 0)
            Button.Size = UDim2.new(0.2, 0, 0.7, 0)
            Button.Font = config.Font
            Button.Text = ""
            
            Button.MouseButton1Click:Connect(function()
                enabled = not enabled
                local toggleAnim = CreateTween(Button, {
                    BackgroundColor3 = enabled and config.AccentColor or config.MainColor
                })
                toggleAnim:Play()
                callback(enabled)
            end)
            
            return Toggle
        end
        
        -- Função para criar Slider
        function tabFunctions:CreateSlider(text, min, max, default, callback)
            local Slider = Instance.new("Frame")
            local Title = Instance.new("TextLabel")
            local SliderBar = Instance.new("Frame")
            local Fill = Instance.new("Frame")
            local Value = Instance.new("TextLabel")
            
            Slider.Name = "Slider"
            Slider.Parent = TabContent
            Slider.BackgroundTransparency = 1
            Slider.Size = UDim2.new(1, -20, 0, 45)
            
            Title.Name = "Title"
            Title.Parent = Slider
            Title.BackgroundTransparency = 1
            Title.Size = UDim2.new(1, 0, 0, 20)
            Title.Font = config.Font
            Title.Text = text
            Title.TextColor3 = config.TextColor
            Title.TextSize = 14
            Title.TextXAlignment = Enum.TextXAlignment.Left
            
            SliderBar.Name = "SliderBar"
            SliderBar.Parent = Slider
            SliderBar.BackgroundColor3 = config.MainColor
            SliderBar.Position = UDim2.new(0, 0, 0.6, 0)
            SliderBar.Size = UDim2.new(0.9, 0, 0, 5)
            
            Fill.Name = "Fill"
            Fill.Parent = SliderBar
            Fill.BackgroundColor3 = config.AccentColor
            Fill.Size = UDim2.new((default - min)/(max - min), 0, 1, 0)
            
            Value.Name = "Value"
            Value.Parent = Slider
            Value.BackgroundTransparency = 1
            Value.Position = UDim2.new(0.9, 10, 0.4, 0)
            Value.Size = UDim2.new(0.1, -10, 0, 20)
            Value.Font = config.Font
            Value.Text = tostring(default)
            Value.TextColor3 = config.TextColor
            Value.TextSize = 14
            
            local function updateSlider(input)
                local pos = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
                local value = math.floor(min + (max - min) * pos)
                Fill.Size = UDim2.new(pos, 0, 1, 0)
                Value.Text = tostring(value)
                callback(value)
            end
            
            SliderBar.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    local connection
                    connection = UserInputService.InputChanged:Connect(function(newInput)
                        if newInput.UserInputType == Enum.UserInputType.MouseMovement then
                            updateSlider(newInput)
                        end
                    end)
                    
                    UserInputService.InputEnded:Connect(function(newInput)
                        if newInput.UserInputType == Enum.UserInputType.MouseButton1 then
                            connection:Disconnect()
                        end
                    end)
                    
                    updateSlider(input)
                end
            end)
            
            return Slider
        end
        
        -- Função para criar Dropdown
        function tabFunctions:CreateDropdown(text, options, callback)
            local Dropdown = Instance.new("Frame")
            local Title = Instance.new("TextLabel")
            local DropButton = Instance.new("TextButton")
            local OptionsFrame = Instance.new("Frame")
            local OptionsList = Instance.new("UIListLayout")
            
            Dropdown.Name = "Dropdown"
            Dropdown.Parent = TabContent
            Dropdown.BackgroundTransparency = 1
            Dropdown.Size = UDim2.new(1, -20, 0, 30)
            
            Title.Name = "Title"
            Title.Parent = Dropdown
            Title.BackgroundTransparency = 1
            Title.Size = UDim2.new(0.7, 0, 1, 0)
            Title.Font = config.Font
            Title.Text = text
            Title.TextColor3 = config.TextColor
            Title.TextSize = 14
            Title.TextXAlignment = Enum.TextXAlignment.Left
            
            DropButton.Name = "DropButton"
            DropButton.Parent = Dropdown
            DropButton.BackgroundColor3 = config.MainColor
            DropButton.Position = UDim2.new(0.7, 0, 0, 0)
            DropButton.Size = UDim2.new(0.3, 0, 1, 0)
            DropButton.Font = config.Font
            DropButton.Text = "▼"
            DropButton.TextColor3 = config.TextColor
            DropButton.TextSize = 14
            
            OptionsFrame.Name = "OptionsFrame"
            OptionsFrame.Parent = Dropdown
            OptionsFrame.BackgroundColor3 = config.MainColor
            OptionsFrame.Position = UDim2.new(0.7, 0, 1, 0)
            OptionsFrame.Size = UDim2.new(0.3, 0, 0, 0)
            OptionsFrame.ClipsDescendants = true
            OptionsFrame.Visible = false
            
            OptionsList.Parent = OptionsFrame
            OptionsList.SortOrder = Enum.SortOrder.LayoutOrder
            
            local function CreateOption(optionText)
                local Option = Instance.new("TextButton")
                Option.Size = UDim2.new(1, 0, 0, 25)
                Option.BackgroundTransparency = 1
                Option.Font = config.Font
                Option.Text = optionText
                Option.TextColor3 = config.TextColor
                Option.TextSize = 14
                Option.Parent = OptionsFrame
                
                Option.MouseButton1Click:Connect(function()
                    DropButton.Text = optionText
                    callback(optionText)
                    local closeAnim = CreateTween(OptionsFrame, {Size = UDim2.new(0.3, 0, 0, 0)})
                    closeAnim:Play()
                    closeAnim.Completed:Wait()
                    OptionsFrame.Visible = false
                end)
            end
            
            for _, option in ipairs(options) do
                CreateOption(option)
            end
            
            local dropdownOpen = false
            DropButton.MouseButton1Click:Connect(function()
                dropdownOpen = not dropdownOpen
                OptionsFrame.Visible = true
                local anim = CreateTween(OptionsFrame, {
                    Size = UDim2.new(0.3, 0, 0, dropdownOpen and OptionsList.AbsoluteContentSize.Y or 0)
                })
                anim:Play()
                if not dropdownOpen then
                    anim.Completed:Wait()
                    OptionsFrame.Visible = false
                end
            end)
            
            return Dropdown
        end
        
        return tabFunctions
    end
    
    -- Minimizar/Maximizar
    local minimized = false
    local MinimizeButton = Instance.new("TextButton")
     MinimizeButton.Position = UDim2.new(1, -25, 0, 0)
    MinimizeButton.Size = UDim2.new(0, 25, 1, 0)
    MinimizeButton.Font = config.Font
    MinimizeButton.Text = "-"
    MinimizeButton.TextColor3 = config.TextColor
    MinimizeButton.TextSize = 20

    MinimizeButton.MouseButton1Click:Connect(function()
        minimized = not minimized
        local anim = CreateTween(Container, {
            Size = minimized and UDim2.new(1, 0, 0, 0) or UDim2.new(1, 0, 1, -30)
        })
        anim:Play()
        MinimizeButton.Text = minimized and "+" or "-"
    end)

    -- Keybind para minimizar (Right Control)
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.RightControl then
            MinimizeButton:Fire("MouseButton1Click")
        end
    end)

    -- Animação de entrada
    Main.Position = UDim2.new(0.5, -250, 0, -400)
    local openAnim = CreateTween(Main, {
        Position = UDim2.new(0.5, -250, 0.5, -150)
    })
    openAnim:Play()

    -- Método para destruir a interface
    function library:Destroy()
        GUI:Destroy()
    end

    return library
end

-- Sistemas adicionais

-- Sistema de Notificações
function Solara:Notify(title, description, duration)
    duration = duration or 3
    
    local Notification = Instance.new("Frame")
    local Title = Instance.new("TextLabel")
    local Description = Instance.new("TextLabel")
    
    Notification.Name = "Notification"
    Notification.Parent = game.CoreGui:FindFirstChild("SolaraLibrary")
    Notification.BackgroundColor3 = config.MainColor
    Notification.Position = UDim2.new(1, 20, 0.8, 0)
    Notification.Size = UDim2.new(0, 250, 0, 80)
    
    Title.Name = "Title"
    Title.Parent = Notification
    Title.BackgroundTransparency = 1
    Title.Position = UDim2.new(0, 10, 0, 5)
    Title.Size = UDim2.new(1, -20, 0, 20)
    Title.Font = config.Font
    Title.Text = title
    Title.TextColor3 = config.TextColor
    Title.TextSize = 16
    Title.TextXAlignment = Enum.TextXAlignment.Left
    
    Description.Name = "Description"
    Description.Parent = Notification
    Description.BackgroundTransparency = 1
    Description.Position = UDim2.new(0, 10, 0, 30)
    Description.Size = UDim2.new(1, -20, 0, 40)
    Description.Font = config.Font
    Description.Text = description
    Description.TextColor3 = config.TextColor
    Description.TextSize = 14
    Description.TextWrapped = true
    Description.TextXAlignment = Enum.TextXAlignment.Left
    
    -- Animação de entrada
    local showAnim = CreateTween(Notification, {
        Position = UDim2.new(1, -270, 0.8, 0)
    })
    showAnim:Play()
    
    -- Timer para remover a notificação
    task.delay(duration, function()
        local hideAnim = CreateTween(Notification, {
            Position = UDim2.new(1, 20, 0.8, 0)
        })
        hideAnim:Play()
        hideAnim.Completed:Wait()
        Notification:Destroy()
    end)
end

-- Sistema de Temas
Solara.Themes = {
    Default = {
        MainColor = Color3.fromRGB(32, 32, 32),
        AccentColor = Color3.fromRGB(0, 120, 255),
        TextColor = Color3.fromRGB(255, 255, 255)
    },
    Dark = {
        MainColor = Color3.fromRGB(25, 25, 25),
        AccentColor = Color3.fromRGB(100, 0, 255),
        TextColor = Color3.fromRGB(255, 255, 255)
    },
    Light = {
        MainColor = Color3.fromRGB(240, 240, 240),
        AccentColor = Color3.fromRGB(0, 100, 200),
        TextColor = Color3.fromRGB(0, 0, 0)
    }
}

function Solara:SetTheme(themeName)
    local theme = self.Themes[themeName]
    if theme then
        config.MainColor = theme.MainColor
        config.AccentColor = theme.AccentColor
        config.TextColor = theme.TextColor
    end
end

return Solara
