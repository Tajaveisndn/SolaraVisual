--[[
    Solara Library
    Version: 1.0.0
    License: MIT
    
    Features:
    - Modern purple theme
    - Smooth animations
    - Fully customizable UI elements
    - Tab system
    - Welcome animation
    - Draggable interface
]]

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local TextService = game:GetService("TextService")

local Solara = {
    Settings = {
        Theme = {
            Primary = Color3.fromRGB(147, 112, 219),    -- Roxo principal
            Secondary = Color3.fromRGB(138, 43, 226),   -- Roxo secundário
            Background = Color3.fromRGB(30, 30, 35),    -- Fundo escuro
            Text = Color3.fromRGB(255, 255, 255),       -- Texto branco
            TextDark = Color3.fromRGB(150, 150, 150),   -- Texto escuro
            BorderColor = Color3.fromRGB(60, 60, 65),   -- Borda escura
            ElementBackground = Color3.fromRGB(40, 40, 45), -- Fundo dos elementos
            ToggleOn = Color3.fromRGB(147, 112, 219),   -- Toggle ativado
            ToggleOff = Color3.fromRGB(100, 100, 100),  -- Toggle desativado
            SliderBackground = Color3.fromRGB(50, 50, 55), -- Fundo do slider
            DropdownBackground = Color3.fromRGB(35, 35, 40) -- Fundo do dropdown
        },
        ToggleKey = Enum.KeyCode.RightControl,
        Logo = "rbxassetid://e8474666919f94a95047a3434c007f4f",
        Name = "Solara Library"
    }
}

-- Utility Functions
local function Create(class, properties)
    local instance = Instance.new(class)
    for property, value in pairs(properties) do
        instance[property] = value
    end
    return instance
end

local function Tween(instance, properties, duration, style, direction)
    local tween = TweenService:Create(
        instance,
        TweenInfo.new(duration or 0.3, style or Enum.EasingStyle.Quad, direction or Enum.EasingDirection.Out),
        properties
    )
    tween:Play()
    return tween
end

local function MakeDraggable(topbar, main)
    local dragging = false
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    topbar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = main.Position

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
            update(input)
        end
    end)
end

-- Ripple Effect Function
local function CreateRipple(parent)
    local ripple = Create("Frame", {
        Parent = parent,
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        BackgroundTransparency = 0.8,
        Position = UDim2.new(0, 0, 0, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Size = UDim2.new(0, 0, 0, 0),
        BorderSizePixel = 0
    })

    local corner = Create("UICorner", {
        Parent = ripple,
        CornerRadius = UDim.new(1, 0)
    })

    return function(input)
        local size = math.max(parent.AbsoluteSize.X, parent.AbsoluteSize.Y) * 1.5
        local pos = UDim2.new(0, input.Position.X - parent.AbsolutePosition.X, 0, input.Position.Y - parent.AbsolutePosition.Y)
        ripple.Position = pos
        
        local tween = Tween(ripple, {
            Size = UDim2.new(0, size, 0, size),
            BackgroundTransparency = 1
        }, 0.5)
        
        tween.Completed:Connect(function()
            ripple.Size = UDim2.new(0, 0, 0, 0)
            ripple.BackgroundTransparency = 0.8
        end)
    end
end

-- Window Creation
function Solara:CreateWindow(config)
    config = config or {}
    local window = {}
    
    -- Main GUI
    local SolaraGui = Create("ScreenGui", {
        Name = "SolaraLibrary",
        Parent = CoreGui,
        IgnoreGuiInset = true,
        ZIndexBehavior = Enum.ZIndexBehavior.Global
    })

    -- Welcome Screen
    local WelcomeFrame = Create("Frame", {
        Name = "Welcome",
        Parent = SolaraGui,
        BackgroundColor3 = self.Settings.Theme.Background,
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Size = UDim2.new(0, 0, 0, 150),
        BorderSizePixel = 0,
        ClipsDescendants = true
    })

    Create("UICorner", {
        Parent = WelcomeFrame,
        CornerRadius = UDim.new(0, 8)
    })

    local WelcomeLogo = Create("ImageLabel", {
        Name = "Logo",
        Parent = WelcomeFrame,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 20, 0.5, -32),
        Size = UDim2.new(0, 64, 0, 64),
        Image = self.Settings.Logo
    })

    local WelcomeTitle = Create("TextLabel", {
        Name = "Title",
        Parent = WelcomeFrame,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 100, 0.5, -15),
        Size = UDim2.new(0, 180, 0, 30),
        Font = Enum.Font.GothamBold,
        Text = "Solara Library",
        TextColor3 = self.Settings.Theme.Text,
        TextSize = 24
    })

    -- Main UI Frame
    local MainFrame = Create("Frame", {
        Name = "Main",
        Parent = SolaraGui,
        BackgroundColor3 = self.Settings.Theme.Background,
        Position = UDim2.new(0.5, 0, 0.5, 0),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Size = UDim2.new(0, 0, 0, 400),
        BorderSizePixel = 0,
        Visible = false,
        ClipsDescendants = true
    })

    Create("UICorner", {
        Parent = MainFrame,
        CornerRadius = UDim.new(0, 8)
    })

    local Topbar = Create("Frame", {
        Name = "Topbar",
        Parent = MainFrame,
        BackgroundColor3 = self.Settings.Theme.Primary,
        Size = UDim2.new(1, 0, 0, 30),
        BorderSizePixel = 0
    })

    Create("UICorner", {
        Parent = Topbar,
        CornerRadius = UDim.new(0, 8)
    })

    local TopbarTitle = Create("TextLabel", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 10, 0, 0),
        Size = UDim2.new(0, 200, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = config.Name or "Solara Hub",
        TextColor3 = self.Settings.Theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    local CloseButton = Create("ImageButton", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -26, 0, 4),
        Size = UDim2.new(0, 22, 0, 22),
        Image = "rbxassetid://6031094678"
    })

    local MinimizeButton = Create("ImageButton", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -52, 0, 4),
        Size = UDim2.new(0, 22, 0, 22),
        Image = "rbxassetid://6031090990"
    })

    -- Welcome Animation
    Tween(WelcomeFrame, {Size = UDim2.new(0, 300, 0, 150)}, 0.5)
    wait(1)
    Tween(WelcomeFrame, {Size = UDim2.new(0, 0, 0, 150)}, 0.5).Completed:Connect(function()
        WelcomeFrame:Destroy()
        MainFrame.Visible = true
        Tween(MainFrame, {Size = UDim2.new(0, 600, 0, 400)}, 0.5)
    end)

    -- Make GUI draggable
    MakeDraggable(Topbar, MainFrame)

    -- Tab System
    local TabContainer = Create("ScrollingFrame", {
        Name = "TabContainer",
        Parent = MainFrame,
        BackgroundColor3 = self.Settings.Theme.ElementBackground,
        Position = UDim2.new(0, 0, 0, 30),
        Size = UDim2.new(0, 150, 1, -30),
        BorderSizePixel = 0,
        ScrollBarThickness = 0,
        CanvasSize = UDim2.new(0, 0, 0, 0),
        ScrollingEnabled = true
    })

    Create("UIListLayout", {
        Parent = TabContainer,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 5)
    })

    local TabContent = Create("Frame", {
        Name = "TabContent",
        Parent = MainFrame,
        BackgroundColor3 = self.Settings.Theme.Background,
        Position = UDim2.new(0, 150, 0, 30),
        Size = UDim2.new(1, -150, 1, -30),
        BorderSizePixel = 0
    })

    -- Close and Minimize Functionality
    CloseButton.MouseButton1Click:Connect(function()
        Tween(MainFrame, {Size = UDim2.new(0, 0, 0, 400)}, 0.5).Completed:Connect(function()
            SolaraGui:Destroy()
        end)
    end)

    local minimized = false
    MinimizeButton.MouseButton1Click:Connect(function()
        minimized = not minimized
        if minimized then
            Tween(MainFrame, {Size = UDim2.new(0, 600, 0, 30)}, 0.5)
        else
            Tween(MainFrame, {Size = UDim2.new(0, 600, 0, 400)}, 0.5)
        end
    end)

    -- Toggle GUI Visibility
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == self.Settings.ToggleKey then
            MainFrame.Visible = not MainFrame.Visible
        end
    end)

    -- Tab Creation Function
    function window:AddTab(name)
        local tab = {}
        local tabSelected = false
        
        local TabButton = Create("TextButton", {
            Name = name,
            Parent = TabContainer,
            BackgroundColor3 = self.Settings.Theme.ElementBackground,
            Size = UDim2.new(1, -10, 0, 35),
            Position = UDim2.new(0, 5, 0, 5),
            Text = name,
            TextColor3 = self.Settings.Theme.Text,
            Font = Enum.Font.Gotham,
            TextSize = 14,
            BorderSizePixel = 0,
            AutoButtonColor = false
        })

        Create("UICorner", {
            Parent = TabButton,
            CornerRadius = UDim.new(0, 6)
        })

        local TabFrame = Create("ScrollingFrame", {
            Name = name.."Content",
            Parent = TabContent,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, -10, 1, -10),
            Position = UDim2.new(0, 5, 0, 5),
            ScrollBarThickness = 2,
            Visible = false,
            CanvasSize = UDim2.new(0, 0, 0, 0)
        })

        Create("UIListLayout", {
            Parent = TabFrame,
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 5)
        })

        -- Tab Selection
        TabButton.MouseButton1Click:Connect(function()
            for _, v in pairs(TabContent:GetChildren()) do
                if v:IsA("ScrollingFrame") then
                    v.Visible = false
                end
            end
            for _, v in pairs(TabContainer:GetChildren()) do
                if v:IsA("TextButton") then
                    Tween(v, {BackgroundColor3 = self.Settings.Theme.ElementBackground})
                end
            end
            TabFrame.Visible = true
            Tween(TabButton, {BackgroundColor3 = self.Settings.Theme.Primary})
        end)

        -- Show first tab by default
        if #TabContainer:GetChildren() == 1 then
            TabFrame.Visible = true
            TabButton.BackgroundColor3 = self.Settings.Theme.Primary
        end

        -- UI Elements
        -- Toggle Element
        function tab:AddToggle(config)
            config = config or {}
            local toggle = {Value = config.Default or false}
            
            local ToggleFrame = Create("Frame", {
                Name = "Toggle",
                Parent = TabFrame,
                BackgroundColor3 = self.Settings.Theme.ElementBackground,
                Size = UDim2.new(1, 0, 0, 40),
                BorderSizePixel = 0
            })

            Create("UICorner", {
                Parent = ToggleFrame,
                CornerRadius = UDim.new(0, 6)
            })

            local ToggleTitle = Create("TextLabel", {
                Parent = ToggleFrame,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 0),
                Size = UDim2.new(0.5, 0, 1, 0),
                Font = Enum.Font.Gotham,
                Text = config.Name or "Toggle",
                TextColor3 = self.Settings.Theme.Text,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local ToggleButton = Create("Frame", {
                Parent = ToggleFrame,
                Position = UDim2.new(1, -50, 0.5, -10),
                Size = UDim2.new(0, 40, 0, 20),
                BackgroundColor3 = self.Settings.Theme.ToggleOff
            })

            Create("UICorner", {
                Parent = ToggleButton,
                CornerRadius = UDim.new(1, 0)
            })

            local ToggleCircle = Create("Frame", {
                Parent = ToggleButton,
                Position = UDim2.new(0, 2, 0.5, -8),
                Size = UDim2.new(0, 16, 0, 16),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255)
            })

            Create("UICorner", {
                Parent = ToggleCircle,
                CornerRadius = UDim.new(1, 0)
            })

            function toggle:Set(value)
                toggle.Value = value
                Tween(ToggleButton, {BackgroundColor3 = value and self.Settings.Theme.ToggleOn or self.Settings.Theme.ToggleOff})
                Tween(ToggleCircle, {Position = value and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)})
                if config.Callback then
                    config.Callback(value)
                end
            end

            ToggleButton.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    toggle:Set(not toggle.Value)
                end
            end)

            toggle:Set(config.Default or false)
            return toggle
        end

        -- Slider Element
        function tab:AddSlider(config)
            config = config or {}
            local slider = {Value = config.Default or config.Min or 0}
            
            local SliderFrame = Create("Frame", {
                Name = "Slider",
                Parent = TabFrame,
                BackgroundColor3 = self.Settings.Theme.ElementBackground,
                Size = UDim2.new(1, 0, 0, 50),
                BorderSizePixel = 0
            })

            Create("UICorner", {
                Parent = SliderFrame,
                CornerRadius = UDim.new(0, 6)
            })

            local SliderTitle = Create("TextLabel", {
                Parent = SliderFrame,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 0),
                Size = UDim2.new(1, -20, 0, 30),
                Font = Enum.Font.Gotham,
                Text = config.Name or "Slider",
                TextColor3 = self.Settings.Theme.Text,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local SliderValue = Create("TextLabel", {
                Parent = SliderFrame,
                BackgroundTransparency = 1,
                Position = UDim2.new(0.9, -10, 0, 0),
                Size = UDim2.new(0.1, 0, 0, 30),
                Font = Enum.Font.Gotham,
                Text = tostring(slider.Value),
                TextColor3 = self.Settings.Theme.TextDark,
                TextSize = 14
            })

            local SliderBar = Create("Frame", {
                Parent = SliderFrame,
                Position = UDim2.new(0, 10, 0, 35),
                Size = UDim2.new(1, -20, 0, 4),
                BackgroundColor3 = self.Settings.Theme.SliderBackground
            })

            Create("UICorner", {
                Parent = SliderBar,
                CornerRadius = UDim.new(1, 0)
            })

            local SliderFill = Create("Frame", {
                Parent = SliderBar,
                Size = UDim2.new(0, 0, 1, 0),
                BackgroundColor3 = self.Settings.Theme.Primary
            })

            Create("UICorner", {
                Parent = SliderFill,
                CornerRadius = UDim.new(1, 0)
            })

            local SliderButton = Create("TextButton", {
                Parent = SliderBar,
                Position = UDim2.new(0, -5, 0.5, -10),
                Size = UDim2.new(0, 20, 0, 20),
                BackgroundColor3 = self.Settings.Theme.Primary,
                Text = "",
                AutoButtonColor = false
            })

            Create("UICorner", {
                Parent = SliderButton,
                CornerRadius = UDim.new(1, 0)
            })

            function slider:Set(value)
                value = math.clamp(value, config.Min or 0, config.Max or 100)
                slider.Value = value
                local percent = (value - (config.Min or 0)) / ((config.Max or 100) - (config.Min or 0))
                
                SliderValue.Text = tostring(math.round(value))
                Tween(SliderFill, {Size = UDim2.new(percent, 0, 1, 0)})
                Tween(SliderButton, {Position = UDim2.new(percent, -10, 0.5, -10)})
                
                if config.Callback then
                    config.Callback(value)
                end
            end

            -- Slider dragging functionality
            local dragging = false
            
            SliderButton.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = true
                end
            end)

            UserInputService.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = false
                end
            end)

            UserInputService.InputChanged:Connect(function(input)
                if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                    local percent = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
                    local value = math.round(((config.Max or 100) - (config.Min or 0)) * percent + (config.Min or 0))
                    slider:Set(value)
                end
            end)

            slider:Set(config.Default or config.Min or 0)
            return slider
        end

        -- Dropdown Element
        function tab:AddDropdown(config)
            config = config or {}
            local dropdown = {Value = config.Default or "", Options = config.Options or {}}
            
            local DropdownFrame = Create("Frame", {
                Name = "Dropdown",
                Parent = TabFrame,
                BackgroundColor3 = self.Settings.Theme.ElementBackground,
                Size = UDim2.new(1, 0, 0, 40),
                ClipsDescendants = true,
                BorderSizePixel = 0
            })

            Create("UICorner", {
                Parent = DropdownFrame,
                CornerRadius = UDim.new(0, 6)
            })

            local DropdownTitle = Create("TextLabel", {
                Parent = DropdownFrame,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 0),
                Size = UDim2.new(1, -40, 0, 40),
                Font = Enum.Font.Gotham,
                Text = config.Name or "Dropdown",
                TextColor3 = self.Settings.Theme.Text,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local DropdownButton = Create("TextButton", {
                Parent = DropdownFrame,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 40),
                Text = "",
                TextColor3 = self.Settings.Theme.Text,
                TextSize = 14
            })

            local DropdownIcon = Create("ImageLabel", {
                Parent = DropdownFrame,
                BackgroundTransparency = 1,
                Position = UDim2.new(1, -30, 0, 10),
                Size = UDim2.new(0, 20, 0, 20),
                Image = "rbxassetid://6031091004",
                ImageColor3 = self.Settings.Theme.Text
            })

            local OptionContainer = Create("Frame", {
                Parent = DropdownFrame,
                Position = UDim2.new(0, 0, 0, 40),
                Size = UDim2.new(1, 0, 0, 0),
                BackgroundTransparency = 1,
                ClipsDescendants = true
            })

            Create("UIListLayout", {
                Parent = OptionContainer,
                SortOrder = Enum.SortOrder.LayoutOrder,
                Padding = UDim.new(0, 5)
            })

            local expanded = false

            function dropdown:Toggle()
                expanded = not expanded
                local optionCount = #dropdown.Options
                Tween(DropdownFrame, {Size = expanded and UDim2.new(1, 0, 0, 45 + (optionCount * 30)) or UDim2.new(1, 0, 0, 40)})
                Tween(DropdownIcon, {Rotation = expanded and 180 or 0})
            end

            function dropdown:Set(value)
                dropdown.Value = value
                DropdownTitle.Text = config.Name .. " - " .. value
                if config.Callback then
                    config.Callback(value)
                end
            end

            function dropdown:Refresh(options)
                dropdown.Options = options
                for _, child in pairs(OptionContainer:GetChildren()) do
                    if child:IsA("TextButton") then
                        child:Destroy()
                    end
                end
                
                for _, option in pairs(options) do
                    local OptionButton = Create("TextButton", {
                        Parent = OptionContainer,
                        Size = UDim2.new(1, 0, 0, 25),
                        BackgroundColor3 = self.Settings.Theme.DropdownBackground,
                        Text = option,
                        TextColor3 = self.Settings.Theme.Text,
                        TextSize = 14,
                        Font = Enum.Font.Gotham,
                        BorderSizePixel = 0
                    })

                    Create("UICorner", {
                        Parent = OptionButton,
                        CornerRadius = UDim.new(0, 6)
                    })

                    OptionButton.MouseButton1Click:Connect(function()
                        dropdown:Set(option)
                        dropdown:Toggle()
                    end)
                end
            end

            DropdownButton.MouseButton1Click:Connect(function()
                dropdown:Toggle()
            end)

            dropdown:Refresh(config.Options or {})
            dropdown:Set(config.Default or "")
            
            return dropdown
        end

        return tab
    end

    return window
end

return Solara
