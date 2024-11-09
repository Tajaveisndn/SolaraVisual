

-- Services
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local Lighting = game:GetService("Lighting")
local Debris = game:GetService("Debris")

-- Constants
local CONFIG_FOLDER = "SolaraConfig"
local CONFIG_FILE = "settings.json"

-- Utility Functions
local function Create(instanceType, properties)
    local instance = Instance.new(instanceType)
    for property, value in pairs(properties) do
        instance[property] = value
    end
    return instance
end

local function DeepCopy(original)
    if typeof(original) ~= "table" then
        return original
    end
    local copy = {}
    for key, value in pairs(original) do
        copy[DeepCopy(key)] = DeepCopy(value)
    end
    return copy
end

-- Solara Table
local Solara = {
    Settings = {
        Theme = {
            Primary = Color3.fromRGB(24, 24, 36),
            Secondary = Color3.fromRGB(30, 30, 45),
            Accent = Color3.fromRGB(44, 120, 224),
            TextColor = Color3.fromRGB(240, 240, 240),
            DarkTextColor = Color3.fromRGB(150, 150, 150),
            ToggleOn = Color3.fromRGB(44, 120, 224),
            ToggleOff = Color3.fromRGB(60, 60, 75)
        },
        Animation = {
            TweenSpeed = 0.3,
            EasingStyle = Enum.EasingStyle.Quart,
            EasingDirection = Enum.EasingDirection.Out,
            TabSwitchSpeed = 0.2,
            MinimizeSpeed = 0.4
        },
        Sound = {
            Enabled = true,
            Volume = 0.5,
            HoverVolume = 0.2
        },
        Keybind = {
            Minimize = Enum.KeyCode.RightControl
        }
    },
    Flags = {},
    SignalManager = {},
    Windows = {},
    IsMinimized = false
}

-- Config System
local function EnsureFolder()
    if not isfolder(CONFIG_FOLDER) then
        makefolder(CONFIG_FOLDER)
    end
end

local function SaveConfig()
    EnsureFolder()
    
    local configData = {
        Theme = Solara.Settings.Theme,
        Sound = Solara.Settings.Sound,
        Keybind = {
            Minimize = tostring(Solara.Settings.Keybind.Minimize)
        },
        Flags = Solara.Flags
    }
    
    writefile(CONFIG_FOLDER .. "/" .. CONFIG_FILE, HttpService:JSONEncode(configData))
end

local function LoadConfig()
    EnsureFolder()
    
    if isfile(CONFIG_FOLDER .. "/" .. CONFIG_FILE) then
        local success, configData = pcall(function()
            return HttpService:JSONDecode(readfile(CONFIG_FOLDER .. "/" .. CONFIG_FILE))
        end)
        
        if success then
            -- Update Theme
            for key, value in pairs(configData.Theme or {}) do
                if typeof(value) == "string" then
                    Solara.Settings.Theme[key] = Color3.fromHex(value)
                else
                    Solara.Settings.Theme[key] = value
                end
            end
            
            -- Update Sound Settings
            for key, value in pairs(configData.Sound or {}) do
                Solara.Settings.Sound[key] = value
            end
            
            -- Update Keybind
            if configData.Keybind and configData.Keybind.Minimize then
                Solara.Settings.Keybind.Minimize = Enum.KeyCode[configData.Keybind.Minimize]
            end
            
            -- Update Flags
            for key, value in pairs(configData.Flags or {}) do
                Solara.Flags[key] = value
            end
        end
    end
end

-- Tween Utility
local function CreateTween(instance, properties, duration, overrideEasing)
    local tweenInfo = TweenInfo.new(
        duration or Solara.Settings.Animation.TweenSpeed,
        overrideEasing and overrideEasing.Style or Solara.Settings.Animation.EasingStyle,
        overrideEasing and overrideEasing.Direction or Solara.Settings.Animation.EasingDirection
    )
    
    return TweenService:Create(instance, tweenInfo, properties)
end

-- Animation Manager
Solara.AnimationManager = {
    ActiveTweens = {},
    
    PlayTween = function(self, instance, properties, duration, overrideEasing)
        if self.ActiveTweens[instance] then
            self.ActiveTweens[instance]:Cancel()
        end
        
        local tween = CreateTween(instance, properties, duration, overrideEasing)
        self.ActiveTweens[instance] = tween
        tween:Play()
        
        tween.Completed:Connect(function()
            self.ActiveTweens[instance] = nil
        end)
        
        return tween
    end,
    
    CancelTweens = function(self, instance)
        if self.ActiveTweens[instance] then
            self.ActiveTweens[instance]:Cancel()
            self.ActiveTweens[instance] = nil
        end
    end
}

-- Theme Manager
function Solara:ChangeUIColor(colorType, color)
    assert(self.Settings.Theme[colorType], "Invalid color type: " .. tostring(colorType))
    self.Settings.Theme[colorType] = color
    
    -- Update all existing UI elements
    for _, window in pairs(self.Windows) do
        window:UpdateTheme()
    end
    
    -- Save the new configuration
    SaveConfig()
end

-- CreateWindow Function
function Solara:CreateWindow(config)
    config = config or {}
    local Window = {}
    
    -- Create Main GUI with Gradient
    local SolaraGui = Create("ScreenGui", {
        Name = "SolaraHub",
        Parent = CoreGui,
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    })

    -- Enhanced Sound System
    local Sounds = {
        Click = Create("Sound", {
            Parent = SolaraGui,
            SoundId = "rbxassetid://6895079853",
            Volume = Solara.Settings.Sound.Volume
        }),
        Hover = Create("Sound", {
            Parent = SolaraGui,
            SoundId = "rbxassetid://6895079853",
            Volume = Solara.Settings.Sound.HoverVolume
        }),
        Switch = Create("Sound", {
            Parent = SolaraGui,
            SoundId = "rbxassetid://6895079853",
            Volume = Solara.Settings.Sound.Volume * 0.8,
            PlaybackSpeed = 1.2
        })
    }

    -- Improved Blur Effect
    local Blur = Create("BlurEffect", {
        Parent = Lighting,
        Enabled = false,
        Size = 0
    })

    -- Main Frame with Gradient
    local Main = Create("Frame", {
        Name = "Main",
        Parent = SolaraGui,
        BackgroundColor3 = Solara.Settings.Theme.Primary,
        BorderSizePixel = 0,
        Position = UDim2.new(0.5, -325, 0.5, -175),
        Size = UDim2.new(0, 650, 0, 350),
        ClipsDescendants = true
    })

    -- Add Gradient to Main
    local MainGradient = Create("UIGradient", {
        Parent = Main,
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(240, 240, 240))
        }),
        Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.97),
            NumberSequenceKeypoint.new(1, 0.95)
        }),
        Rotation = 45
    })

    -- Improved Corner and Shadow
    Create("UICorner", {
        Parent = Main,
        CornerRadius = UDim.new(0, 8)
    })

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

    -- Enhanced Top Bar with Gradient
    local TopBar = Create("Frame", {
        Name = "TopBar",
        Parent = Main,
        BackgroundColor3 = Solara.Settings.Theme.Secondary,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 35)
    })

    local TopBarGradient = Create("UIGradient", {
        Parent = TopBar,
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(240, 240, 240))
        }),
        Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.97),
            NumberSequenceKeypoint.new(1, 0.95)
        })
    })

    Create("UICorner", {
        Parent = TopBar,
        CornerRadius = UDim.new(0, 8)
    })

    -- Enhanced Title with Accent
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

    -- Improved Close Button
    local CloseButton = Create("ImageButton", {
        Parent = TopBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -30, 0.5, -8),
        Size = UDim2.new(0, 16, 0, 16),
        Image = "rbxassetid://6035047409",
        ImageColor3 = Solara.Settings.Theme.TextColor
    })

    -- Modern Tab System
    local TabContainer = Create("ScrollingFrame", {
        Parent = Main,
        BackgroundColor3 = Solara.Settings.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 10, 0, 45),
        Size = UDim2.new(0, 130, 1, -55),
        CanvasSize = UDim2.new(0, 0, 0, 0),
        ScrollBarThickness = 2,
        ScrollBarImageColor3 = Solara.Settings.Theme.Accent
    })

    -- Add Smooth Scrolling to TabContainer
    local function smoothScroll(frame)
        local targetPosition = frame.CanvasPosition
        local currentPosition = frame.CanvasPosition
        
        RunService.RenderStepped:Connect(function(delta)
            if (currentPosition - targetPosition).Magnitude > 1 then
                currentPosition = currentPosition:Lerp(targetPosition, delta * 10)
                frame.CanvasPosition = currentPosition
            end
        end)
        
        frame.Changed:Connect(function(prop)
            if prop == "CanvasPosition" then
                targetPosition = frame.CanvasPosition
            end
        end)
    end

    smoothScroll(TabContainer)

    -- Tab Container Styling
    Create("UICorner", {
        Parent = TabContainer,
        CornerRadius = UDim.new(0, 8)
    })

    local TabList = Create("UIListLayout", {
        Parent = TabContainer,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 5)
    })

    -- Tab Animation System
    local TabAnimations = {
        selectedTab = nil,
        indicators = {},
        
        createIndicator = function(self, tabButton)
            local indicator = Create("Frame", {
                Parent = tabButton,
                BackgroundColor3 = Solara.Settings.Theme.Accent,
                Position = UDim2.new(0, 0, 1, -2),
                Size = UDim2.new(0, 0, 0, 2),
                BorderSizePixel = 0,
                Name = "Indicator"
            })
            
            Create("UICorner", {
                Parent = indicator,
                CornerRadius = UDim.new(1, 0)
            })
            
            self.indicators[tabButton] = indicator
            return indicator
        end,
        
        selectTab = function(self, tabButton)
            -- Deselect previous tab
            if self.selectedTab then
                Solara.AnimationManager:PlayTween(
                    self.indicators[self.selectedTab],
                    {Size = UDim2.new(0, 0, 0, 2)},
                    0.3
                )
            end
            
            -- Select new tab
            self.selectedTab = tabButton
            Solara.AnimationManager:PlayTween(
                self.indicators[tabButton],
                {Size = UDim2.new(1, -20, 0, 2)},
                0.3
            )
        end
    }

    -- Content Container with Gradient
    local ContentContainer = Create("Frame", {
        Parent = Main,
        BackgroundColor3 = Solara.Settings.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 150, 0, 45),
        Size = UDim2.new(1, -160, 1, -55)
    })

    Create("UICorner", {
        Parent = ContentContainer,
        CornerRadius = UDim.new(0, 8)
    })

    Create("UIGradient", {
        Parent = ContentContainer,
        Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(240, 240, 240))
        }),
        Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0.97),
            NumberSequenceKeypoint.new(1, 0.95)
        })
    })

    -- Enhanced Tab Creation
    function Window:CreateTab(name)
        local Tab = {}
        
        -- Create modern tab button
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

        -- Add hover effect gradient
        local TabGradient = Create("UIGradient", {
            Parent = TabButton,
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
                ColorSequenceKeypoint.new(1, Color3.fromRGB(240, 240, 240))
            }),
            Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.97),
                NumberSequenceKeypoint.new(1, 0.95)
            })
        })

        Create("UICorner", {
            Parent = TabButton,
            CornerRadius = UDim.new(0, 6)
        })

        -- Create indicator
        TabAnimations:createIndicator(TabButton)

        -- Create tab content with smooth transitions
        local TabContent = Create("ScrollingFrame", {
            Parent = ContentContainer,
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 1, 0),
            CanvasSize = UDim2.new(0, 0, 0, 0),
            ScrollBarThickness = 2,
            ScrollBarImageColor3 = Solara.Settings.Theme.Accent,
            Visible = false
        })

        smoothScroll(TabContent)

        -- UIListLayout for TabContent
        Create("UIListLayout", {
            Parent = TabContent,
            HorizontalAlignment = Enum.HorizontalAlignment.Center,
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 5)
        })

        -- Enhanced tab switching animation
        TabButton.MouseButton1Click:Connect(function()
            -- Sound effect
            if Solara.Settings.Sound.Enabled then
                Sounds.Switch:Play()
            end

            -- Visual updates
            TabAnimations:selectTab(TabButton)

            -- Content transition
            for _, content in pairs(ContentContainer:GetChildren()) do
                if content:IsA("ScrollingFrame") then
                    if content ~= TabContent then
                        -- Fade out other content
                        Solara.AnimationManager:PlayTween(content, {
                            BackgroundTransparency = 1,
                            ScrollBarImageTransparency = 1
                        }, 0.2)
                        wait(0.2)
                        content.Visible = false
                    end
                end
            end

            -- Show and fade in new content
            TabContent.BackgroundTransparency = 1
            TabContent.ScrollBarImageTransparency = 1
            TabContent.Visible = true
            Solara.AnimationManager:PlayTween(TabContent, {
                BackgroundTransparency = 0,
                ScrollBarImageTransparency = 0
            }, 0.2)
        end)

        -- UI Elements for Tab
        function Tab:CreateButton(text, callback)
            local ButtonContainer = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 38),
                ClipsDescendants = true
            })

            Create("UICorner", {
                Parent = ButtonContainer,
                CornerRadius = UDim.new(0, 6)
            })

            local Button = Create("TextButton", {
                Parent = ButtonContainer,
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 1, 0),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13
            })

            -- Ripple Effect
            local function CreateRipple(x, y)
                local Ripple = Create("Frame", {
                    Parent = ButtonContainer,
                    BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                    BackgroundTransparency = 0.7,
                    Position = UDim2.new(0, x, 0, y),
                    Size = UDim2.new(0, 0, 0, 0),
                    AnchorPoint = Vector2.new(0.5, 0.5),
                })

                Create("UICorner", {
                    Parent = Ripple,
                    CornerRadius = UDim.new(1, 0)
                })

                local targetSize = UDim2.new(0, ButtonContainer.AbsoluteSize.X * 1.5, 0, ButtonContainer.AbsoluteSize.X * 1.5)
                
                Solara.AnimationManager:PlayTween(Ripple, {
                    Size = targetSize,
                    BackgroundTransparency = 1
                }, 0.5)

                Debris:AddItem(Ripple, 0.5)
            end

            Button.MouseButton1Down:Connect(function(x, y)
                local relativeX = x - ButtonContainer.AbsolutePosition.X
                local relativeY = y - ButtonContainer.AbsolutePosition.Y
                CreateRipple(relativeX, relativeY)
                
                if Solara.Settings.Sound.Enabled then
                    Sounds.Click:Play()
                end
                
                callback()
            end)

            -- Hover Effect
            Button.MouseEnter:Connect(function()
                Solara.AnimationManager:PlayTween(ButtonContainer, {
                    BackgroundColor3 = Solara.Settings.Theme.Primary:Lerp(Color3.new(1,1,1), 0.05)
                }, 0.2)
                
                if Solara.Settings.Sound.Enabled then
                    Sounds.Hover:Play()
                end
            end)

            Button.MouseLeave:Connect(function()
                Solara.AnimationManager:PlayTween(ButtonContainer, {
                    BackgroundColor3 = Solara.Settings.Theme.Primary
                }, 0.2)
            end)

            return ButtonContainer
        end

        function Tab:CreateToggle(text, default, callback)
            local ToggleContainer = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 38)
            })

            Create("UICorner", {
                Parent = ToggleContainer,
                CornerRadius = UDim.new(0, 6)
            })

            local Label = Create("TextLabel", {
                Parent = ToggleContainer,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 15, 0, 0),
                Size = UDim2.new(1, -65, 1, 0),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local ToggleButton = Create("Frame", {
                Parent = ToggleContainer,
                BackgroundColor3 = default and Solara.Settings.Theme.ToggleOn or Solara.Settings.Theme.ToggleOff,
                Position = UDim2.new(1, -52, 0.5, -12),
                Size = UDim2.new(0, 42, 0, 24),
                Name = "ToggleButton"
            })

            Create("UICorner", {
                Parent = ToggleButton,
                CornerRadius = UDim.new(1, 0)
            })

            local Circle = Create("Frame", {
                Parent = ToggleButton,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Position = default and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10),
                Size = UDim2.new(0, 20, 0, 20),
                Name = "Circle"
            })

            Create("UICorner", {
                Parent = Circle,
                CornerRadius = UDim.new(1, 0)
            })

            -- Add shadow to circle
            local CircleShadow = Create("ImageLabel", {
                Parent = Circle,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -2, 0, -2),
                Size = UDim2.new(1, 4, 1, 4),
                Image = "rbxassetid://6014261993",
                ImageColor3 = Color3.fromRGB(0, 0, 0),
                ImageTransparency = 0.5,
                ScaleType = Enum.ScaleType.Slice,
                SliceCenter = Rect.new(49, 49, 450, 450)
            })

            local toggled = default
            local isAnimating = false

            local function updateToggle()
                if isAnimating then return end
                isAnimating = true
                toggled = not toggled

                -- Animate the background color
                Solara.AnimationManager:PlayTween(ToggleButton, {
                    BackgroundColor3 = toggled and Solara.Settings.Theme.ToggleOn or Solara.Settings.Theme.ToggleOff
                }, 0.3)

                -- Animate the circle with bounce effect
                local targetPosition = toggled and UDim2.new(1, -22, 0.5, -10) or UDim2.new(0, 2, 0.5, -10)
                
                -- First move slightly past the target
                local overshootPosition = toggled and UDim2.new(1, -20, 0.5, -10) or UDim2.new(0, 0, 0.5, -10)
                Solara.AnimationManager:PlayTween(Circle, {
                    Position = overshootPosition
                }, 0.2, {Style = Enum.EasingStyle.Quad, Direction = Enum.EasingDirection.Out})
                
                wait(0.2)
                
                -- Then bounce back to the actual target
                Solara.AnimationManager:PlayTween(Circle, {
                    Position = targetPosition
                }, 0.1, {Style = Enum.EasingStyle.Quad, Direction = Enum.EasingDirection.InOut})
                
                if Solara.Settings.Sound.Enabled then
                    Sounds.Switch:Play()
                end

                callback(toggled)
                isAnimating = false
            end

            ToggleContainer.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    updateToggle()
                end
            end)

            -- Hover Effect
            ToggleContainer.MouseEnter:Connect(function()
                Solara.AnimationManager:PlayTween(ToggleContainer, {
                    BackgroundColor3 = Solara.Settings.Theme.Primary:Lerp(Color3.new(1,1,1), 0.05)
                }, 0.2)
                
                if Solara.Settings.Sound.Enabled then
                    Sounds.Hover:Play()
                end
            end)

            ToggleContainer.MouseLeave:Connect(function()
                Solara.AnimationManager:PlayTween(ToggleContainer, {
                    BackgroundColor3 = Solara.Settings.Theme.Primary
                }, 0.2)
            end)

            return {
                Instance = ToggleContainer,
                SetValue = function(value)
                    if toggled ~= value then
                        updateToggle()
                    end
                end,
                GetValue = function()
                    return toggled
                end
            }
        end

        function Tab:CreateSlider(text, min, max, default, callback)
            local SliderContainer = Create("Frame", {
                Parent = TabContent,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 54)
            })

            Create("UICorner", {
                Parent = SliderContainer,
                CornerRadius = UDim.new(0, 6)
            })

            local Title = Create("TextLabel", {
                Parent = SliderContainer,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 15, 0, 5),
                Size = UDim2.new(1, -120, 0, 20),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local Value = Create("TextBox", {
                Parent = SliderContainer,
                BackgroundColor3 = Solara.Settings.Theme.Secondary,
                Position = UDim2.new(1, -105, 0, 5),
                Size = UDim2.new(0, 90, 0, 20),
                Font = Enum.Font.GothamSemibold,
                Text = tostring(default),
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                ClipsDescendants = true,
                TextEditable = false
            })

            Create("UICorner", {
                Parent = Value,
                CornerRadius = UDim.new(0, 4)
            })

            local SliderBar = Create("Frame", {
                Parent = SliderContainer,
                BackgroundColor3 = Solara.Settings.Theme.Secondary,
                Position = UDim2.new(0, 15, 0, 35),
                Size = UDim2.new(1, -30, 0, 4),
                Name = "SliderBar"
            })

            Create("UICorner", {
                Parent = SliderBar,
                CornerRadius = UDim.new(1, 0)
            })

            local Fill = Create("Frame", {
                Parent = SliderBar,
                BackgroundColor3 = Solara.Settings.Theme.Accent,
                Size = UDim2.new((default - min)/(max - min), 0, 1, 0),
                Name = "Fill"
            })

            Create("UICorner", {
                Parent = Fill,
                CornerRadius = UDim.new(1, 0)
            })

            local Knob = Create("Frame", {
                Parent = SliderBar,
                BackgroundColor3 = Solara.Settings.Theme.Accent,
                Position = UDim2.new((default - min)/(max - min), -6, 0.5, -6),
                Size = UDim2.new(0, 12, 0, 12),
                ZIndex = 2,
                Name = "Knob"
            })

            Create("UICorner", {
                Parent = Knob,
                CornerRadius = UDim.new(1, 0)
            })

            local KnobShadow = Create("ImageLabel", {
                Parent = Knob,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -2, 0, -2),
                Size = UDim2.new(1, 4, 1, 4),
                Image = "rbxassetid://6014261993",
                ImageColor3 = Color3.fromRGB(0, 0, 0),
                ImageTransparency = 0.5,
                ScaleType = Enum.ScaleType.Slice,
                SliceCenter = Rect.new(49, 49, 450, 450),
                ZIndex = 1
            })

            -- Slider functionality with improved animation
            local dragging = false
            local function updateSlider(input)
                local pos = math.clamp((input.Position.X - SliderBar.AbsolutePosition.X) / SliderBar.AbsoluteSize.X, 0, 1)
                local value = math.floor(min + ((max - min) * pos))
                
                -- Animate Fill and Knob
                Solara.AnimationManager:PlayTween(Fill, {
                    Size = UDim2.new(pos, 0, 1, 0)
                }, 0.1)
                
                Solara.AnimationManager:PlayTween(Knob, {
                    Position = UDim2.new(pos, -6, 0.5, -6)
                }, 0.1)
                
                Value.Text = tostring(value)
                callback(value)

                -- Hover effect on knob while dragging
                Knob.Size = UDim2.new(0, 14, 0, 14)
                Knob.Position = UDim2.new(pos, -7, 0.5, -7)
            end

            SliderBar.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = true
                    updateSlider(input)
                end
            end)

            UserInputService.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                    dragging = false
                    -- Reset knob size
                    Solara.AnimationManager:PlayTween(Knob, {
                        Size = UDim2.new(0, 12, 0, 12)
                    }, 0.1)
                    local pos = (tonumber(Value.Text) - min)/(max - min)
                    Solara.AnimationManager:PlayTween(Knob, {
                        Position = UDim2.new(pos, -6, 0.5, -6)
                    }, 0.1)
                end
            end)

            UserInputService.InputChanged:Connect(function(input)
                if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                    updateSlider(input)
                end
            end)

            -- Value input box functionality
            Value.FocusLost:Connect(function(enterPressed)
                if enterPressed then
                    local newValue = tonumber(Value.Text)
                    if newValue then
                        newValue = math.clamp(newValue, min, max)
                        Value.Text = tostring(newValue)
                        
                        local pos = (newValue - min)/(max - min)
                        Solara.AnimationManager:PlayTween(Fill, {
                            Size = UDim2.new(pos, 0, 1, 0)
                        }, 0.1)
                        
                        Solara.AnimationManager:PlayTween(Knob, {
                            Position = UDim2.new(pos, -6, 0.5, -6)
                        }, 0.1)
                        
                        callback(newValue)
                    else
                        Value.Text = tostring(default)
                    end
                end
            end)

            return {
                Instance = SliderContainer,
                SetValue = function(value)
                    value = math.clamp(value, min, max)
                    Value.Text = tostring(value)
                    local pos = (value - min)/(max - min)
                    Solara.AnimationManager:PlayTween(Fill, {
                        Size = UDim2.new(pos, 0, 1, 0)
                    }, 0.1)
                    Solara.AnimationManager:PlayTween(Knob, {
                        Position = UDim2.new(pos, -6, 0.5, -6)
                    }, 0.1)
                    callback(value)
                end,
                GetValue = function()
                    return tonumber(Value.Text) or default
                end
            }
        end

        return Tab
    end

    -- Settings Panel
    function Window:CreateSettings()
        local SettingsTab = self:CreateTab("Settings")
        
        -- UI Color Settings
        SettingsTab:CreateLabel = function(text)
            local Label = Create("TextLabel", {
                Parent = SettingsTab.Instance,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 15, 0, 10),
                Size = UDim2.new(1, -30, 0, 20),
                Font = Enum.Font.GothamBold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 14,
                TextXAlignment = Enum.TextXAlignment.Left
            })
        end

        SettingsTab:CreateLabel("UI Colors")
        
        local colorOptions = {
            {"Primary", "Main background color"},
            {"Secondary", "Secondary elements color"},
            {"Accent", "Accent elements color"},
            {"TextColor", "Main text color"},
            {"DarkTextColor", "Secondary text color"},
            {"ToggleOn", "Toggle enabled color"},
            {"ToggleOff", "Toggle disabled color"}
        }

        for _, colorOption in ipairs(colorOptions) do
            SettingsTab:CreateColorPicker(colorOption[1], colorOption[2], Solara.Settings.Theme[colorOption[1]], function(color)
                Solara.Settings.Theme[colorOption[1]] = color
                SaveConfig()
                Solara:ChangeUIColor(colorOption[1], color)
            end)
        end

        -- Animation Settings
        SettingsTab:CreateLabel("Animation")
        
        SettingsTab:CreateSlider("Animation Speed", 0.1, 1, Solara.Settings.Animation.TweenSpeed, function(value)
            Solara.Settings.Animation.TweenSpeed = value
            SaveConfig()
        end)

        -- Sound Settings
        SettingsTab:CreateLabel("Sound")
        
        SettingsTab:CreateToggle("Enable Sounds", Solara.Settings.Sound.Enabled, function(value)
            Solara.Settings.Sound.Enabled = value
            SaveConfig()
        end)

        SettingsTab:CreateSlider("Sound Volume", 0, 1, Solara.Settings.Sound.Volume, function(value)
            Solara.Settings.Sound.Volume = value
            for _, sound in pairs(Sounds) do
                sound.Volume = value
            end
            SaveConfig()
        end)

        -- Keybind Settings
        SettingsTab:CreateLabel("Keybinds")
        
        SettingsTab:CreateKeybind = function(text, default, callback)
            local KeybindContainer = Create("Frame", {
                Parent = SettingsTab.Instance,
                BackgroundColor3 = Solara.Settings.Theme.Primary,
                Size = UDim2.new(1, 0, 0, 38)
            })

            Create("UICorner", {
                Parent = KeybindContainer,
                CornerRadius = UDim.new(0, 6)
            })

            local Label = Create("TextLabel", {
                Parent = KeybindContainer,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 15, 0, 0),
                Size = UDim2.new(1, -135, 1, 0),
                Font = Enum.Font.GothamSemibold,
                Text = text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                TextXAlignment = Enum.TextXAlignment.Left
            })

            local KeybindButton = Create("TextButton", {
                Parent = KeybindContainer,
                BackgroundColor3 = Solara.Settings.Theme.Secondary,
                Position = UDim2.new(1, -120, 0.5, -15),
                Size = UDim2.new(0, 100, 0, 30),
                Font = Enum.Font.GothamSemibold,
                Text = default and default.Name or "None",
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                AutoButtonColor = false
            })

            Create("UICorner", {
                Parent = KeybindButton,
                CornerRadius = UDim.new(0, 4)
            })

            -- Add subtle shadow
            local KeybindShadow = Create("ImageLabel", {
                Parent = KeybindButton,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -2, 0, -2),
                Size = UDim2.new(1, 4, 1, 4),
                Image = "rbxassetid://6014261993",
                ImageColor3 = Color3.fromRGB(0, 0, 0),
                ImageTransparency = 0.8,
                ScaleType = Enum.ScaleType.Slice,
                SliceCenter = Rect.new(49, 49, 450, 450),
                ZIndex = 1
            })

            local currentKey = default
            local isBinding = false

            local function updateValue(value)
                currentKey = value
                KeybindButton.Text = currentKey and currentKey.Name or "None"
                if callback then
                    callback(currentKey)
                end
            end

            -- Binding animation
            local function startBinding()
                isBinding = true
                local dots = ""
                local connection
                
                -- Animate dots while binding
                connection = RunService.Heartbeat:Connect(function()
                    if not isBinding then
                        connection:Disconnect()
                        return
                    end
                    dots = dots .. "."
                    if #dots > 3 then dots = "" end
                    KeybindButton.Text = "Press Key" .. dots
                end)
                
                -- Pulse animation
                local function doPulse()
                    if not isBinding then return end
                    
                    Solara.AnimationManager:PlayTween(KeybindButton, {
                        BackgroundColor3 = Solara.Settings.Theme.Accent,
                        TextColor3 = Color3.fromRGB(255, 255, 255)
                    }, 0.5)
                    
                    wait(0.5)
                    
                    if isBinding then
                        Solara.AnimationManager:PlayTween(KeybindButton, {
                            BackgroundColor3 = Solara.Settings.Theme.Secondary,
                            TextColor3 = Solara.Settings.Theme.TextColor
                        }, 0.5)
                    end
                    
                    if isBinding then
                        doPulse()
                    end
                end
                
                doPulse()
            end

            local function stopBinding(newKey)
                isBinding = false
                
                if newKey then
                    updateValue(newKey)
                    
                    -- Save to config
                    Solara.Flags[text .. "_Keybind"] = newKey.Name
                    SaveConfig()
                else
                    KeybindButton.Text = currentKey and currentKey.Name or "None"
                end
                
                Solara.AnimationManager:PlayTween(KeybindButton, {
                    BackgroundColor3 = Solara.Settings.Theme.Secondary,
                    TextColor3 = Solara.Settings.Theme.TextColor
                }, 0.2)
            end

            KeybindButton.MouseButton1Click:Connect(function()
                if not isBinding then
                    if Solara.Settings.Sound.Enabled then
                        Sounds.Click:Play()
                    end
                    startBinding()
                    
                    local inputConnection
                    inputConnection = UserInputService.InputBegan:Connect(function(input)
                        if input.UserInputType == Enum.UserInputType.Keyboard then
                            inputConnection:Disconnect()
                            stopBinding(input.KeyCode)
                        elseif input.UserInputType == Enum.UserInputType.MouseButton1 or
                               input.UserInputType == Enum.UserInputType.MouseButton2 then
                            inputConnection:Disconnect()
                            stopBinding(nil)
                        end
                    end)
                end
            end)

            -- Hover effects
            KeybindButton.MouseEnter:Connect(function()
                if not isBinding then
                    Solara.AnimationManager:PlayTween(KeybindButton, {
                        BackgroundColor3 = Solara.Settings.Theme.Secondary:Lerp(Color3.new(1,1,1), 0.1)
                    }, 0.2)
                end
                
                if Solara.Settings.Sound.Enabled then
                    Sounds.Hover:Play()
                end
            end)

            KeybindButton.MouseLeave:Connect(function()
                if not isBinding then
                    Solara.AnimationManager:PlayTween(KeybindButton, {
                        BackgroundColor3 = Solara.Settings.Theme.Secondary
                    }, 0.2)
                end
            end)

            return {
                Instance = KeybindContainer,
                SetKey = updateValue,
                GetKey = function() return currentKey end
            }
        end

        SettingsTab:CreateKeybind("Toggle UI", Solara.Settings.Keybind.Minimize, function(key)
            Solara.Settings.Keybind.Minimize = key
            SaveConfig()
        end)

        -- Config Management
        SettingsTab:CreateLabel("Configuration")
        
        SettingsTab:CreateButton("Save Configuration", function()
            SaveConfig()
            Solara.NotificationSystem:CreateNotification("Success", "Configuration saved successfully!", 3, "success")
        end)

        SettingsTab:CreateButton("Reset to Default", function()
            -- Create confirmation dialog
            local dialog = SettingsTab:CreateConfirmDialog("Reset Configuration", 
                "Are you sure you want to reset all settings to default? This cannot be undone.", 
                function(confirmed)
                    if confirmed then
                        -- Reset all settings to default
                        Solara.Settings = DeepCopy({
                            Theme = {
                                Primary = Color3.fromRGB(24, 24, 36),
                                Secondary = Color3.fromRGB(30, 30, 45),
                                Accent = Color3.fromRGB(44, 120, 224),
                                TextColor = Color3.fromRGB(240, 240, 240),
                                DarkTextColor = Color3.fromRGB(150, 150, 150),
                                ToggleOn = Color3.fromRGB(44, 120, 224),
                                ToggleOff = Color3.fromRGB(60, 60, 75)
                            },
                            Animation = {
                                TweenSpeed = 0.3,
                                EasingStyle = Enum.EasingStyle.Quart,
                                EasingDirection = Enum.EasingDirection.Out,
                                TabSwitchSpeed = 0.2,
                                MinimizeSpeed = 0.4
                            },
                            Sound = {
                                Enabled = true,
                                Volume = 0.5,
                                HoverVolume = 0.2
                            },
                            Keybind = {
                                Minimize = Enum.KeyCode.RightControl
                            }
                        })
                        SaveConfig()
                        Solara:ChangeUIColor("Primary", Solara.Settings.Theme.Primary)
                        Solara:ChangeUIColor("Secondary", Solara.Settings.Theme.Secondary)
                        Solara:ChangeUIColor("Accent", Solara.Settings.Theme.Accent)
                        Solara:ChangeUIColor("TextColor", Solara.Settings.Theme.TextColor)
                        Solara:ChangeUIColor("DarkTextColor", Solara.Settings.Theme.DarkTextColor)
                        Solara:ChangeUIColor("ToggleOn", Solara.Settings.Theme.ToggleOn)
                        Solara:ChangeUIColor("ToggleOff", Solara.Settings.Theme.ToggleOff)
                        Window:UpdateTheme()
                        Solara.NotificationSystem:CreateNotification("Success", "Settings reset to default!", 3, "success")
                    end
                end
            )
            dialog:Show()
        end)
    end

    -- Additional UI Elements for Settings
    function Tab:CreateColorPicker(text, description, default, callback)
        local ColorPickerContainer = Create("Frame", {
            Parent = TabContent,
            BackgroundColor3 = Solara.Settings.Theme.Primary,
            Size = UDim2.new(1, 0, 0, 70)
        })

        Create("UICorner", {
            Parent = ColorPickerContainer,
            CornerRadius = UDim.new(0, 6)
        })

        local Title = Create("TextLabel", {
            Parent = ColorPickerContainer,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 15, 0, 5),
            Size = UDim2.new(1, -30, 0, 20),
            Font = Enum.Font.GothamSemibold,
            Text = text,
            TextColor3 = Solara.Settings.Theme.TextColor,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left
        })

        local Description = Create("TextLabel", {
            Parent = ColorPickerContainer,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 15, 0, 25),
            Size = UDim2.new(1, -30, 0, 20),
            Font = Enum.Font.Gotham,
            Text = description,
            TextColor3 = Solara.Settings.Theme.DarkTextColor,
            TextSize = 12,
            TextXAlignment = Enum.TextXAlignment.Left
        })

        local ColorDisplay = Create("Frame", {
            Parent = ColorPickerContainer,
            BackgroundColor3 = default,
            Position = UDim2.new(1, -55, 0, 10),
            Size = UDim2.new(0, 40, 0, 40)
        })

        Create("UICorner", {
            Parent = ColorDisplay,
            CornerRadius = UDim.new(0, 4)
        })

        -- Color picker popup and functionality would go here
        -- This is a simplified version for demonstration
        ColorDisplay.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                -- Here you would implement the color picker popup
                -- For now, we'll just cycle through some preset colors
                local colors = {
                    Color3.fromRGB(255, 0, 0),
                    Color3.fromRGB(0, 255, 0),
                    Color3.fromRGB(0, 0, 255),
                    Color3.fromRGB(255, 255, 0),
                    Color3.fromRGB(255, 0, 255),
                    Color3.fromRGB(0, 255, 255)
                }
                
                local currentColor = ColorDisplay.BackgroundColor3
                local nextColorIndex = 1
                
                for i, color in ipairs(colors) do
                    if currentColor == color then
                        nextColorIndex = (i % #colors) + 1
                        break
                    end
                end
                
                local newColor = colors[nextColorIndex]
                Solara.AnimationManager:PlayTween(ColorDisplay, {
                    BackgroundColor3 = newColor
                }, 0.3)
                
                callback(newColor)
            end
        end)

        return ColorDisplay
    end

    -- Initialize window
    Solara.AnimationManager:PlayTween(Blur, {Size = 10}, 0.3)
    
    -- Add window to Solara
    table.insert(Solara.Windows, Window)
    
    -- Create Settings
    Window:CreateSettings()
    
    return Window
end

-- Notification System
Solara.NotificationSystem = {
    Notifications = {},
    MaxNotifications = 5,
    
    Setup = function(self)
        self.Container = Create("Frame", {
            Parent = CoreGui:FindFirstChild("SolaraHub"),
            BackgroundTransparency = 1,
            Position = UDim2.new(1, -330, 0, 20),
            Size = UDim2.new(0, 300, 1, -40),
            Name = "NotificationContainer"
        })
        
        Create("UIListLayout", {
            Parent = self.Container,
            HorizontalAlignment = Enum.HorizontalAlignment.Right,
            SortOrder = Enum.SortOrder.LayoutOrder,
            Padding = UDim.new(0, 10)
        })
    end,
    
    CreateNotification = function(self, title, message, duration, type)
        if #self.Notifications >= self.MaxNotifications then
            self.Notifications[1]:Destroy()
            table.remove(self.Notifications, 1)
        end
        
        -- Notification types: success, warning, error, info
        local colors = {
            success = Color3.fromRGB(46, 204, 113),
            warning = Color3.fromRGB(241, 196, 15),
            error = Color3.fromRGB(231, 76, 60),
            info = Solara.Settings.Theme.Accent
        }
        
        local icons = {
            success = "rbxassetid://7733715400",
            warning = "rbxassetid://7733715400",
            error = "rbxassetid://7733715400",
            info = "rbxassetid://7733715400"
        }
        
        local Notification = Create("Frame", {
            Parent = self.Container,
            BackgroundColor3 = Solara.Settings.Theme.Primary,
            Size = UDim2.new(1, 0, 0, 0), -- Start with 0 height
            BackgroundTransparency = 1,
            ClipsDescendants = true
        })
        
        Create("UICorner", {
            Parent = Notification,
            CornerRadius = UDim.new(0, 8)
        })
        
        -- Add glass blur effect
        local BlurEffect = Create("ImageLabel", {
            Parent = Notification,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 1, 0),
            Image = "rbxassetid://6014261993",
            ImageColor3 = Color3.fromRGB(255, 255, 255),
            ImageTransparency = 0.9,
            ScaleType = Enum.ScaleType.Slice,
            SliceCenter = Rect.new(49, 49, 450, 450)
        })
        
        local Icon = Create("ImageLabel", {
            Parent = Notification,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 15, 0, 15),
            Size = UDim2.new(0, 20, 0, 20),
            Image = icons[type] or icons.info,
            ImageColor3 = colors[type] or colors.info
        })
        
        local Title = Create("TextLabel", {
            Parent = Notification,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 45, 0, 10),
            Size = UDim2.new(1, -60, 0, 20),
            Font = Enum.Font.GothamBold,
            Text = title,
            TextColor3 = Solara.Settings.Theme.TextColor,
            TextSize = 14,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        
        local Message = Create("TextLabel", {
            Parent = Notification,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 45, 0, 30),
            Size = UDim2.new(1, -60, 0, 40),
            Font = Enum.Font.Gotham,
            Text = message,
            TextColor3 = Solara.Settings.Theme.DarkTextColor,
            TextSize = 13,
            TextWrapped = true,
            TextXAlignment = Enum.TextXAlignment.Left
        })
        
        -- Progress bar
        local ProgressBar = Create("Frame", {
            Parent = Notification,
            BackgroundColor3 = colors[type] or colors.info,
            Position = UDim2.new(0, 0, 1, -2),
            Size = UDim2.new(0, 0, 0, 2)
        })

        -- Animate notification entrance
        Solara.AnimationManager:PlayTween(Notification, {
            Size = UDim2.new(1, 0, 0, 80),
            BackgroundTransparency = 0
        }, 0.3, {Style = Enum.EasingStyle.Back})

        -- Animate progress bar
        Solara.AnimationManager:PlayTween(ProgressBar, {
            Size = UDim2.new(1, 0, 0, 2)
        }, duration or 5)

        table.insert(self.Notifications, Notification)
        
        -- Remove notification after duration
        delay(duration or 5, function()
            if Notification and Notification.Parent then
                Solara.AnimationManager:PlayTween(Notification, {
                    Size = UDim2.new(1, 0, 0, 0),
                    BackgroundTransparency = 1
                }, 0.3, {Style = Enum.EasingStyle.Back, Direction = Enum.EasingDirection.In}).Completed:Wait()
                Notification:Destroy()
                
                for i, v in pairs(self.Notifications) do
                    if v == Notification then
                        table.remove(self.Notifications, i)
                        break
                    end
                end
            end
        end)
    end
}

-- Tooltip System
Solara.TooltipSystem = {
    ActiveTooltip = nil,
    
    CreateTooltip = function(self, parent, text)
        local Tooltip = Create("Frame", {
            Parent = CoreGui:FindFirstChild("SolaraHub"),
            BackgroundColor3 = Solara.Settings.Theme.Primary,
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 200, 0, 30),
            Visible = false,
            ZIndex = 100
        })
        
        Create("UICorner", {
            Parent = Tooltip,
            CornerRadius = UDim.new(0, 6)
        })
        
        local TextLabel = Create("TextLabel", {
            Parent = Tooltip,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 0),
            Size = UDim2.new(1, -20, 1, 0),
            Font = Enum.Font.Gotham,
            Text = text,
            TextColor3 = Solara.Settings.Theme.TextColor,
            TextSize = 12,
            TextWrapped = true,
            ZIndex = 101
        })
        
        -- Show/Hide events
        parent.MouseEnter:Connect(function()
            if self.ActiveTooltip then
                self.ActiveTooltip.Visible = false
            end
            
            self.ActiveTooltip = Tooltip
            Tooltip.Position = UDim2.new(0, parent.AbsolutePosition.X + parent.AbsoluteSize.X + 10,
                                       0, parent.AbsolutePosition.Y)
            Tooltip.Visible = true
            
            Solara.AnimationManager:PlayTween(Tooltip, {
                BackgroundTransparency = 0
            }, 0.2)
        end)
        
        parent.MouseLeave:Connect(function()
            Solara.AnimationManager:PlayTween(Tooltip, {
                BackgroundTransparency = 1
            }, 0.2).Completed:Wait()
            
            if self.ActiveTooltip == Tooltip then
                self.ActiveTooltip = nil
                Tooltip.Visible = false
            end
        end)
    end
}

-- Context Menu System
Solara.ContextMenuSystem = {
    CreateContextMenu = function(self, items)
        local ContextMenu = Create("Frame", {
            Parent = CoreGui:FindFirstChild("SolaraHub"),
            BackgroundColor3 = Solara.Settings.Theme.Primary,
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 150, 0, #items * 30 + 10),
            Visible = false,
            ZIndex = 1000
        })
        
        Create("UICorner", {
            Parent = ContextMenu,
            CornerRadius = UDim.new(0, 6)
        })
        
        -- Add items
        for i, item in ipairs(items) do
            local Button = Create("TextButton", {
                Parent = ContextMenu,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 5, 0, (i-1) * 30 + 5),
                Size = UDim2.new(1, -10, 0, 25),
                Font = Enum.Font.Gotham,
                Text = item.Text,
                TextColor3 = Solara.Settings.Theme.TextColor,
                TextSize = 13,
                ZIndex = 1001
            })
            
            -- Hover effect
            Button.MouseEnter:Connect(function()
                Solara.AnimationManager:PlayTween(Button, {
                    BackgroundTransparency = 0,
                    BackgroundColor3 = Solara.Settings.Theme.Accent
                }, 0.2)
            end)
            
            Button.MouseLeave:Connect(function()
                Solara.AnimationManager:PlayTween(Button, {
                    BackgroundTransparency = 1
                }, 0.2)
            end)
            
            Button.MouseButton1Click:Connect(function()
                if item.Callback then
                    item.Callback()
                end
                ContextMenu.Visible = false
            end)
        end
        
        -- Show/Hide context menu
        return {
            Show = function(position)
                ContextMenu.Position = position
                ContextMenu.Visible = true
                Solara.AnimationManager:PlayTween(ContextMenu, {
                    BackgroundTransparency = 0
                }, 0.2)
            end,
            Hide = function()
                Solara.AnimationManager:PlayTween(ContextMenu, {
                    BackgroundTransparency = 1
                }, 0.2).Completed:Wait()
                ContextMenu.Visible = false
            end
        }
    end
}

-- Window Minimize Animation
function Solara:InitializeWindowMinimize(Window)
    Window:UpdateTheme = function()
        -- Update theme for window elements if needed
        -- This function can be expanded based on UI elements
    end

    Window.IsMinimized = false

    function Window:MinimizeToggle()
        local main = self.Instance.Main
        local isMinimized = not self.IsMinimized
        self.IsMinimized = isMinimized
        
        if isMinimized then
            -- Store the current position before minimizing
            self.LastPosition = main.Position
            
            -- Animate minimize
            Solara.AnimationManager:PlayTween(main, {
                Size = UDim2.new(0, 650, 0, 35),
                Position = UDim2.new(1, -670, 1, -55)
            }, Solara.Settings.Animation.MinimizeSpeed, {Style = Enum.EasingStyle.Back, Direction = Enum.EasingDirection.In})
        else
            -- Animate restore
            Solara.AnimationManager:PlayTween(main, {
                Size = UDim2.new(0, 650, 0, 350),
                Position = self.LastPosition
            }, Solara.Settings.Animation.MinimizeSpeed, {Style = Enum.EasingStyle.Back, Direction = Enum.EasingDirection.Out})
        end
    end

    -- Connect CloseButton to minimize toggle
    local CloseButton = self.Instance.TopBar.CloseButton
    CloseButton.MouseButton1Click:Connect(function()
        Window:MinimizeToggle()
    end)
end

-- Notification and Tooltip Setup
function Solara:Initialize()
    -- Initialize all systems
    self.NotificationSystem:Setup()
    
    -- Initialize Tooltip System (if needed)
    -- self.TooltipSystem:Setup() -- Assuming a setup function if necessary

    -- Initialize Context Menu System (if needed)
    -- self.ContextMenuSystem:Setup() -- Assuming a setup function if necessary

    -- Setup global keybind
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == self.Settings.Keybind.Minimize then
            for _, window in pairs(self.Windows) do
                window:MinimizeToggle()
            end
        end
    end)
    
    -- Load saved configuration
    LoadConfig()
    
    -- Create welcome notification
    self.NotificationSystem:CreateNotification(
        "Welcome to SolaraUI",
        "Press " .. self.Settings.Keybind.Minimize.Name .. " to toggle the UI",
        5,
        "info"
    )
end

-- Initialize Notification and Tooltip Systems
Solara.NotificationSystem:Setup()

-- Example of creating a window (You can remove or modify this as needed)
local mainWindow = Solara:CreateWindow({Title = "Main Window"})
Solara:Initialize()

-- Return the Solara table
return Solara
