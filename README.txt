--[[
    SolaraUI Library
    Author: Seu Nome
    Description: Uma moderna e profissional UI library para Roblox exploits
    Version: 1.0.0
]]

-- Services
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")

-- Variables
local Solara = {
    Settings = {
        Theme = {
            Primary = Color3.fromRGB(24, 24, 36),
            Secondary = Color3.fromRGB(30, 30, 45),
            Accent = Color3.fromRGB(44, 120, 224),
            TextColor = Color3.fromRGB(240, 240, 240),
            DarkTextColor = Color3.fromRGB(150, 150, 150)
        },
        Animation = {
            TweenSpeed = 0.3,
            EasingStyle = Enum.EasingStyle.Quart,
            EasingDirection = Enum.EasingDirection.Out
        },
        UseSound = true
    },
    Flags = {},
    SignalManager = {},
    Windows = {}
}

-- Utility Functions
local function CreateTween(instance, properties, duration)
    local tween = TweenService:Create(
        instance,
        TweenInfo.new(
            duration or Solara.Settings.Animation.TweenSpeed,
            Solara.Settings.Animation.EasingStyle,
            Solara.Settings.Animation.EasingDirection
        ),
        properties
    )
    return tween
end

local function Create(instanceType, properties)
    local instance = Instance.new(instanceType)
    for property, value in pairs(properties) do
        instance[property] = value
    end
    return instance
end

local function MakeClickable(button, color, hoverColor, clickSound)
    button.MouseEnter:Connect(function()
        CreateTween(button, {BackgroundColor3 = hoverColor}):Play()
    end)
    
    button.MouseLeave:Connect(function()
        CreateTween(button, {BackgroundColor3 = color}):Play()
    end)
    
    button.MouseButton1Down:Connect(function()
        CreateTween(button, {BackgroundColor3 = color:Lerp(Color3.new(0,0,0), 0.2)}):Play()
        if Solara.Settings.UseSound and clickSound then
            clickSound:Play()
        end
    end)
    
    button.MouseButton1Up:Connect(function()
        CreateTween(button, {BackgroundColor3 = hoverColor}):Play()
    end)
end

function Solara:CreateWindow(config)
    config = config or {}
    local Window = {}
    
    -- Create Main GUI
    local SolaraGui = Create("ScreenGui", {
        Name = "SolaraHub",
        Parent = CoreGui,
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    })

    -- Sounds
    local ClickSound = Create("Sound", {
        Parent = SolaraGui,
        SoundId = "rbxassetid://6895079853",
        Volume = 0.5
    })

    local HoverSound = Create("Sound", {
        Parent = SolaraGui,
        SoundId = "rbxassetid://6895079853",
        Volume = 0.2
    })

    -- Create Blur Effect
    local Blur = Create("BlurEffect", {
        Parent = game:GetService("Lighting"),
        Enabled = false,
        Size = 0
    })

    -- Main Frame
    local Main = Create("Frame", {
        Name = "Main",
        Parent = SolaraGui,
        BackgroundColor3 = Solara.Settings.Theme.Primary,
        BorderSizePixel = 0,
        Position = UDim2.new(0.5, -325, 0.5, -175),
        Size = UDim2.new(0, 650, 0, 350),
        ClipsDescendants = true
    })

    -- Add Main Corner
    local MainCorner = Create("UICorner", {
        Parent = Main,
        CornerRadius = UDim.new(0, 8)
    })

    -- Add Shadow
    local Shadow = Create("ImageLabel", {
        Parent = Main,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, -15, 0, -15),
        Size = UDim2.new(1, 30, 1, 30),
        Image = "rbxassetid://6014261993",
        ImageColor3 = Color3.fromRGB(0, 0, 0),
        ImageTransparency = 0.5,
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(49, 49, 450, 450)
    })

    -- Top Bar
    local TopBar = Create("Frame", {
        Name = "TopBar",
        Parent = Main,
        BackgroundColor3 = Solara.Settings.Theme.Secondary,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 35)
    })

    -- Add Top Bar Corner
    Create("UICorner", {
        Parent = TopBar,
        CornerRadius = UDim.new(0, 8)
    })

    -- Title
    local Title = Create("TextLabel", {
        Parent = TopBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 15, 0, 0),
        Size = UDim2.new(0, 200, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = config.Title or "SolaraHub",
        TextColor3 = Solara.Settings.Theme.TextColor,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    -- Close Button
    local CloseButton = Create("TextButton", {
        Parent = TopBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -30, 0, 5),
        Size = UDim2.new(0, 25, 0, 25),
        Font = Enum.Font.GothamBold,
        Text = "×",
        TextColor3 = Solara.Settings.Theme.TextColor,
        TextSize = 20
    })

    -- Add hover effect to close button
    CloseButton.MouseEnter:Connect(function()
        CreateTween(CloseButton, {TextColor3 = Color3.fromRGB(255, 95, 95)}):Play()
        if Solara.Settings.UseSound then
            HoverSound:Play()
        end
    end)

    CloseButton.MouseLeave:Connect(function()
        CreateTween(CloseButton, {TextColor3 = Solara.Settings.Theme.TextColor}):Play()
    end)

    -- Add click effect to close button
    CloseButton.MouseButton1Click:Connect(function()
        if Solara.Settings.UseSound then
            ClickSound:Play()
        end
        
        -- Fade out animation
        CreateTween(Main, {BackgroundTransparency = 1}):Play()
        CreateTween(Shadow, {ImageTransparency = 1}):Play()
        CreateTween(Blur, {Size = 0}):Play()
        
        wait(0.3)
        SolaraGui:Destroy()
        Blur:Destroy()
    end)

    -- Make window draggable
    local Dragging = false
    local DragInput
    local DragStart
    local StartPos

    TopBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Dragging = true
            DragStart = input.Position
            StartPos = Main.Position
        end
    end)

    TopBar.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            Dragging = false
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement and Dragging then
            local Delta = input.Position - DragStart
            Main.Position = UDim2.new(StartPos.X.Scale, StartPos.X.Offset + Delta.X, StartPos.Y.Scale, StartPos.Y.Offset + Delta.Y)
        end
    end)

    -- Create Tab Container and Content Container
    local TabContainer = Create("ScrollingFrame", {
        Parent = Main,
        BackgroundColor3 = Solara.Settings.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 10, 0, 45),
        Size = UDim2.new(0, 130, 1, -55),
        CanvasSize = UDim2.new(0, 0, 0, 0),
        ScrollBarThickness = 0
    })

    -- Add corners to TabContainer
    Create("UICorner", {
        Parent = TabContainer,
        CornerRadius = UDim.new(0, 8)
    })

    -- Add list layout to TabContainer
    local TabList = Create("UIListLayout", {
        Parent = TabContainer,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 5)
    })

    local ContentContainer = Create("Frame", {
        Parent = Main,
        BackgroundColor3 = Solara.Settings.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 150, 0, 45),
        Size = UDim2.new(1, -160, 1, -55)
    })

    -- Add corners to ContentContainer
    Create("UICorner", {
        Parent = ContentContainer,
        CornerRadius = UDim.new(0, 8)
    })

    -- Tab Functions
    function Window:CreateTab(name)
        local Tab = {}
        
        -- Create tab button
        local TabButton = Create("TextButton", {
            Parent = TabContainer,
            BackgroundColor3 = Solara.Settings.Theme.Secondary,
            BorderSizePixel = 0,
            Size = UDim2.new(0.9, 0, 0, 32),
            Font = Enum.Font.GothamSemibold,
            Text = name,
            TextColor3 = Solara.Settings.Theme.DarkTextColor,
            TextSize = 13,
            AutoButtonColor = false
        })

        -- Add corners to tab button
        Create("UICorner", {
            Parent = TabButton,
            CornerRadius = UDim.new(0, 6)
        })

        -- Create tab content
        local TabContent = Create("ScrollingFrame", {
            Parent = ContentContainer,
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 1, 0),
            CanvasSize = UDim2.new(0, 0, 0, 0),
            ScrollBarThickness = 3,
            Visible = false
        })

        -- Add padding to tab content
        local ContentPadding = Create("UIPadding", {
            Parent = TabContent,
            PaddingLeft = UDim.new(0, 10),
            PaddingRight = UDim.new(0, 10),
            PaddingTop = UDim.new(0, 10)
        })

        -- Add list layout to tab content
        local ContentList = Create("UIListLayout", {
            Parent = TabContent,
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 8)
        })

        -- Update canvas size when content changes
        ContentList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            TabContent.CanvasSize = UDim2.new(0, 0, 0, ContentList.AbsoluteContentSize.Y + 20)
        end)

        -- Tab button functionality
        TabButton.MouseButton1Click:Connect(function()
            for _, button in pairs(TabContainer:GetChildren()) do
                if button:IsA("TextButton") then
                    CreateTween(button, {
                        BackgroundColor3 = Solara.Settings.Theme.Secondary,
                        TextColor3 = Solara.Settings.Theme.DarkTextColor
                    }):Play()
                end
            end
            
            CreateTween(TabButton, {
                BackgroundColor3 = Solara.Settings.Theme.Accent,
                TextColor3 = Solara.Settings.Theme.TextColor
            }):Play()

            for _, content in pairs(ContentContainer:GetChildren()) do
                if content:IsA("ScrollingFrame") then
                    content.Visible = false
                end
            end
            TabContent.Visible = true
            
            if Solara.Settings.UseSound then
                ClickSound:Play()
            end
        end)

        -- Tab Element Functions
        function Tab:CreateButton(text, callback)
            local Button = Create("TextButton", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 32),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                AutoButtonColor = false
            })

            Create("UICorner", {
                Parent = Button,
                CornerRadius = UDim.new(0, 6)
            })

            MakeClickable(
                Button, 
                Solara.Settings.Theme.Primary, 
                Solara.Settings.Theme.Primary:Lerp(Color3.new(1,1,1), 0.05),
                ClickSound
            )

            Button.MouseButton1Click:Connect(function()
                callback()
            end)
            
            return Button
        end

        function Tab:CreateToggle(text, default, callback)
            local Toggle = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 32)
            })

            Create("UICorner", {
                Parent = Toggle,
                CornerRadius = UDim.new(0, 6)
            })

            local ToggleButton = Create("TextButton", {
                Parent = Toggle,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 1, 0),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left,
                TextTruncate = Enum.TextTruncate.AtEnd
            })

            Create("UIPadding", {
                Parent = ToggleButton,
                PaddingLeft = UDim.new(0, 10)
            })

            local ToggleIndicator = Create("Frame", {
                Parent = Toggle,
BackgroundColor3 = default and Solara.Settings.Theme.Accent or Solara.Settings.Theme.Primary,
                Position = UDim2.new(1, -42, 0.5, -10),
                Size = UDim2.new(0, 32, 0, 20)
            })

            Create("UICorner", {
                Parent = ToggleIndicator,
                CornerRadius = UDim.new(1, 0)
            })

            local Circle = Create("Frame", {
                Parent = ToggleIndicator,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Position = default and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8),
                Size = UDim2.new(0, 16, 0, 16)
            })

            Create("UICorner", {
                Parent = Circle,
                CornerRadius = UDim.new(1, 0)
            })

            local toggled = default
            ToggleButton.MouseButton1Click:Connect(function()
                toggled = not toggled
                
                if toggled then
                    CreateTween(ToggleIndicator, {BackgroundColor3 = Solara.Settings.Theme.Accent}):Play()
                    CreateTween(Circle, {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
                else
                    CreateTween(ToggleIndicator, {BackgroundColor3 = Solara.Settings.Theme.Primary}):Play()
                    CreateTween(Circle, {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
                end
                
                if Solara.Settings.UseSound then
                    ClickSound:Play()
                end
                
                callback(toggled)
            end)
            
            return Toggle
        end

        function Tab:CreateSlider(text, min, max, default, callback)
            local Slider = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 50)
            })

            Create("UICorner", {
                Parent = Slider,
                CornerRadius = UDim.new(0, 6)
            })

            local Title = Create("TextLabel", {
                Parent = Slider,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 5),
                Size = UDim2.new(1, -20, 0, 20),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local Value = Create("TextLabel", {
                Parent = Slider,
                BackgroundTransparency = 1,
                Position = UDim2.new(1, -60, 0, 5),
                Size = UDim2.new(0, 50, 0, 20),
                Font = Enum.Font.GothamSemibold,
                Text = tostring(default),
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Right
            })

            local SliderBar = Create("Frame", {
                Parent = Slider,
                BackgroundColor3 = Solara.Settings.Theme.Secondary,
                Position = UDim2.new(0, 10, 0, 35),
                Size = UDim2.new(1, -20, 0, 5)
            })

            Create("UICorner", {
                Parent = SliderBar,
                CornerRadius = UDim.new(1, 0)
            })

            local Fill = Create("Frame", {
                Parent = SliderBar,
                BackgroundColor3 = Solara.Settings.Theme.Accent,
                Size = UDim2.new((default - min)/(max - min), 0, 1, 0)
            })

            Create("UICorner", {
                Parent = Fill,
                CornerRadius = UDim.new(1, 0)
            })

            local dragging = false
            SliderBar.InputBegan:Connect(function(input)
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
                    local pos = UDim2.new(math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1), 0, 1, 0)
                    Fill.Size = pos
                    local value = math.floor(min + ((max - min) * pos.X.Scale))
                    Value.Text = tostring(value)
                    callback(value)
                end
            end)
            
            return Slider
        end

        function Tab:CreateDropdown(text, options, default, callback)
            local Dropdown = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                ClipsDescendants = true,
                Size = UDim2.new(1, 0, 0, 32)
            })

            Create("UICorner", {
                Parent = Dropdown,
                CornerRadius = UDim.new(0, 6)
            })

            local DropdownButton = Create("TextButton", {
                Parent = Dropdown,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 32),
                Font = Enum.Font.GothamSemibold,
                Text = text .. ": " .. tostring(default),
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13
            })

            local OptionContainer = Create("Frame", {
                Parent = Dropdown,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 0, 0, 32),
                Size = UDim2.new(1, 0, 0, #options * 32)
            })

            local isOpen = false
            local function toggleDropdown()
                isOpen = not isOpen
                local size = isOpen and UDim2.new(1, 0, 0, 32 + (#options * 32)) or UDim2.new(1, 0, 0, 32)
                CreateTween(Dropdown, {Size = size}):Play()
            end

            DropdownButton.MouseButton1Click:Connect(toggleDropdown)

            for i, option in pairs(options) do
                local OptionButton = Create("TextButton", {
                    Parent = OptionContainer,
                    BackgroundColor3 = Solara.Settings.Theme.Secondary,
                    Position = UDim2.new(0, 0, 0, (i-1) * 32),
                    Size = UDim2.new(1, 0, 0, 32),
                    Font = Enum.Font.GothamSemibold,
                    Text = tostring(option),
                    TextColor3 = Solara.Settings.Theme.TextColor,
                    TextSize = 13
                })

                Create("UICorner", {
                    Parent = OptionButton,
                    CornerRadius = UDim.new(0, 6)
                })

                OptionButton.MouseButton1Click:Connect(function()
                    DropdownButton.Text = text .. ": " .. tostring(option)
                    toggleDropdown()
                    callback(option)
                    
                    if Solara.Settings.UseSound then
                        ClickSound:Play()
                    end
                end)
            end
            
            return Dropdown
        end

        -- Select first tab by default
        if #TabContainer:GetChildren() == 2 then
            TabButton.BackgroundColor3 = Solara.Settings.Theme.Accent
            TabButton.TextColor3 = Solara.Settings.Theme.TextColor
            TabContent.Visible = true
        end

        return Tab
    end

    -- Show window with animation
    CreateTween(Blur, {Size = 10}):Play()
    
    return Window
end

return Solara
