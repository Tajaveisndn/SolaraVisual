--[[
    SolaraUI Library - Minimal Version
    Authors: ny0xdel4s and fearqwzz_
    Discord: ny0xdel4s e fearqwzz_
    Version: 1.0.0
]]

-- Services
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")

local SolaraUI = {
    Theme = {
        Primary = Color3.fromRGB(24, 24, 36),
        Secondary = Color3.fromRGB(30, 30, 45),
        Accent = Color3.fromRGB(44, 120, 224),
        Text = Color3.fromRGB(240, 240, 240),
        Toggle = Color3.fromRGB(44, 120, 224)
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

local function CreateTween(instance, props, duration)
    return TweenService:Create(
        instance,
        TweenInfo.new(duration or 0.3, Enum.EasingStyle.Quart),
        props
    )
end

function SolaraUI:CreateWindow(title)
    local Window = {}
    
    -- Main GUI
    local ScreenGui = Create("ScreenGui", {
        Name = "SolaraUI",
        Parent = CoreGui,
        ResetOnSpawn = false
    })
    
    -- Main Frame
    local Main = Create("Frame", {
        Name = "Main",
        Parent = ScreenGui,
        BackgroundColor3 = self.Theme.Primary,
        BorderSizePixel = 0,
        Position = UDim2.new(0.5, -200, 0.5, -150),
        Size = UDim2.new(0, 400, 0, 300),
        ClipsDescendants = true
    })
    
    Create("UICorner", {
        Parent = Main,
        CornerRadius = UDim.new(0, 8)
    })
    
    -- Top Bar
    local TopBar = Create("Frame", {
        Name = "TopBar",
        Parent = Main,
        BackgroundColor3 = self.Theme.Secondary,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 35)
    })
    
    Create("UICorner", {
        Parent = TopBar,
        CornerRadius = UDim.new(0, 8)
    })
    
    -- Title
    Create("TextLabel", {
        Parent = TopBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 15, 0, 0),
        Size = UDim2.new(1, -30, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = title or "SolaraUI",
        TextColor3 = self.Theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left
    })
    
    -- Content Frame
    local Content = Create("Frame", {
        Parent = Main,
        BackgroundColor3 = self.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 10, 0, 45),
        Size = UDim2.new(1, -20, 1, -55)
    })
    
    Create("UICorner", {
        Parent = Content,
        CornerRadius = UDim.new(0, 8)
    })
    
    local ElementList = Create("UIListLayout", {
        Parent = Content,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 5)
    })
    
    -- Toggle Function
    function Window:CreateToggle(text, default, callback)
        local ToggleFrame = Create("Frame", {
            Parent = Content,
            BackgroundColor3 = self.Theme.Primary,
            Size = UDim2.new(1, -20, 0, 35),
            BorderSizePixel = 0
        })
        
        Create("UICorner", {
            Parent = ToggleFrame,
            CornerRadius = UDim.new(0, 6)
        })
        
        local Label = Create("TextLabel", {
            Parent = ToggleFrame,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 0),
            Size = UDim2.new(1, -60, 1, 0),
            Font = Enum.Font.Gotham,
            Text = text,
            TextColor3 = self.Theme.Text,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        
        local Switch = Create("Frame", {
            Parent = ToggleFrame,
            BackgroundColor3 = default and self.Theme.Toggle or self.Theme.Secondary,
            Position = UDim2.new(1, -50, 0.5, -10),
            Size = UDim2.new(0, 40, 0, 20)
        })
        
        Create("UICorner", {
            Parent = Switch,
            CornerRadius = UDim.new(1, 0)
        })
        
        local Circle = Create("Frame", {
            Parent = Switch,
            BackgroundColor3 = Color3.fromRGB(255, 255, 255),
            Position = default and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8),
            Size = UDim2.new(0, 16, 0, 16)
        })
        
        Create("UICorner", {
            Parent = Circle,
            CornerRadius = UDim.new(1, 0)
        })
        
        local Toggled = default or false
        
        ToggleFrame.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                Toggled = not Toggled
                
                CreateTween(Circle, {
                    Position = Toggled and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
                }):Play()
                
                CreateTween(Switch, {
                    BackgroundColor3 = Toggled and self.Theme.Toggle or self.Theme.Secondary
                }):Play()
                
                if callback then
                    callback(Toggled)
                end
            end
        end)
    end
    
    -- Slider Function
    function Window:CreateSlider(text, min, max, default, callback)
        local SliderFrame = Create("Frame", {
            Parent = Content,
            BackgroundColor3 = self.Theme.Primary,
            Size = UDim2.new(1, -20, 0, 50),
            BorderSizePixel = 0
        })
        
        Create("UICorner", {
            Parent = SliderFrame,
            CornerRadius = UDim.new(0, 6)
        })
        
        local Label = Create("TextLabel", {
            Parent = SliderFrame,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 5),
            Size = UDim2.new(1, -20, 0, 20),
            Font = Enum.Font.Gotham,
            Text = text,
            TextColor3 = self.Theme.Text,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        
        local SliderBack = Create("Frame", {
            Parent = SliderFrame,
            BackgroundColor3 = self.Theme.Secondary,
            Position = UDim2.new(0, 10, 0, 35),
            Size = UDim2.new(1, -20, 0, 4)
        })
        
        Create("UICorner", {
            Parent = SliderBack,
            CornerRadius = UDim.new(1, 0)
        })
        
        local SliderFill = Create("Frame", {
            Parent = SliderBack,
            BackgroundColor3 = self.Theme.Accent,
            Size = UDim2.new((default - min)/(max - min), 0, 1, 0)
        })
        
        Create("UICorner", {
            Parent = SliderFill,
            CornerRadius = UDim.new(1, 0)
        })
        
        local ValueLabel = Create("TextLabel", {
            Parent = SliderFrame,
            BackgroundTransparency = 1,
            Position = UDim2.new(1, -50, 0, 5),
            Size = UDim2.new(0, 40, 0, 20),
            Font = Enum.Font.Gotham,
            Text = tostring(default),
            TextColor3 = self.Theme.Text,
            TextSize = 13
        })
        
        local IsDragging = false
        
        SliderBack.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                IsDragging = true
            end
        end)
        
        game:GetService("UserInputService").InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                IsDragging = false
            end
        end)
        
        game:GetService("UserInputService").InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseMovement and IsDragging then
                local pos = game:GetService("UserInputService"):GetMouseLocation()
                local relative = (pos.X - SliderBack.AbsolutePosition.X) / SliderBack.AbsoluteSize.X
                relative = math.clamp(relative, 0, 1)
                
                local value = math.floor(min + ((max - min) * relative))
                ValueLabel.Text = tostring(value)
                
                CreateTween(SliderFill, {
                    Size = UDim2.new(relative, 0, 1, 0)
                }):Play()
                
                if callback then
                    callback(value)
                end
            end
        end)
    end

    -- Notify Function
    function Window:Notify(title, text, duration)
        duration = duration or 3
        
        local Notification = Create("Frame", {
            Parent = ScreenGui,
            BackgroundColor3 = self.Theme.Primary,
            Position = UDim2.new(1, -320, 1, -90),
            Size = UDim2.new(0, 300, 0, 80),
            BorderSizePixel = 0
        })
        
        Create("UICorner", {
            Parent = Notification,
            CornerRadius = UDim.new(0, 8)
        })
        
        local Title = Create("TextLabel", {
            Parent = Notification,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 15, 0, 10),
            Size = UDim2.new(1, -30, 0, 20),
            Font = Enum.Font.GothamBold,
            Text = title,
            TextColor3 = self.Theme.Text,
            TextSize = 14,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        
        local Message = Create("TextLabel", {
            Parent = Notification,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 15, 0, 35),
            Size = UDim2.new(1, -30, 0, 35),
            Font = Enum.Font.Gotham,
            Text = text,
            TextColor3 = self.Theme.Text,
            TextSize = 13,
            TextWrapped = true,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        
        -- Animation
        CreateTween(Notification, {
            Position = UDim2.new(1, -320, 1, -100)
        }):Play()
        
        wait(duration)
        
        CreateTween(Notification, {
            Position = UDim2.new(1, 0, 1, -100)
        }):Play()
        
        wait(0.5)
        Notification:Destroy()
    end
    
    -- Change UI Color Function
    function Window:ChangeColor(colorType, color)
        self.Theme[colorType] = color
    end
    
    return Window
end

return SolaraUI
