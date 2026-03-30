--[[
    OrionUI - Professional Roblox UI Library
    Inspired by the best features from top Roblox UI libraries
    
    Features:
    - Modern gamesense-inspired design
    - Glow effects (configurable)
    - Smooth animations
    - Tab system
    - Config save/load
    - Notifications
]]

local OrionUI = {}

-- Services
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local GuiService = game:GetService("GuiService")

local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- Theme (customizable)
OrionUI.Theme = {
    Accent = Color3.fromRGB(255, 120, 30),
    AccentGradient = Color3.fromRGB(255, 140, 50),
    Background = Color3.fromRGB(25, 25, 25),
    Background2 = Color3.fromRGB(18, 18, 20),
    Background3 = Color3.fromRGB(12, 12, 12),
    Text = Color3.fromRGB(205, 205, 205),
    SubText = Color3.fromRGB(130, 130, 130),
    Outline = Color3.fromRGB(12, 12, 12),
    Element = Color3.fromRGB(35, 35, 35),
    SectionBackground = Color3.fromRGB(22, 22, 24),
    SectionInner = Color3.fromRGB(15, 15, 17),
    SectionTop = Color3.fromRGB(18, 18, 20),
    Danger = Color3.fromRGB(255, 60, 60),
    Success = Color3.fromRGB(60, 255, 120),
    GlowEnabled = true,
    GlowIntensity = 0.6
}

OrionUI.Flags = {}

-- Font - Using Code font like skeet UI
local function GetFont()
    return Font.new("rbxassetid://12187287426", Enum.FontWeight.SemiBold, Enum.FontStyle.Normal)
end

-- Utility Functions
local function CreateInstance(class, props)
    local inst = Instance.new(class)
    for k, v in pairs(props) do
        inst[k] = v
    end
    return inst
end

local function Round(instance, radius)
    local corner = CreateInstance("UICorner", {
        CornerRadius = UDim.new(0, radius),
        Parent = instance
    })
    return corner
end

local function AddGlow(instance, color, intensity)
    if not OrionUI.Theme.GlowEnabled then return end
    
    local glow = CreateInstance("ImageLabel", {
        Parent = instance,
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 20, 1, 20),
        Position = UDim2.new(0, -10, 0, -10),
        Image = "rbxassetid://14077986724",
        ImageColor3 = color or OrionUI.Theme.Accent,
        ImageTransparency = 1 - (intensity or OrionUI.Theme.GlowIntensity),
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(20, 20, 20, 20),
        ZIndex = instance.ZIndex - 1
    })
    return glow
end

local function AddDraggable(frame, handle)
    local dragging = false
    local dragInput = nil
    local dragStart = nil
    local startPos = nil

    local function Update(input)
        if not dragging then return end
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    (handle or frame).InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position
            
            local connection
            connection = input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                    connection:Disconnect()
                end
            end)
        end
    end)

    (handle or frame).InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            Update(input)
        end
    end)
end

-- Window Creation
function OrionUI:Window(options)
    options = options or {}
    local Window = {
        Name = options.Name or "OrionUI",
        SubName = options.SubName or "Premium UI Library",
        Logo = options.Logo or "rbxassetid://13390337711",
        Accent = options.Accent or OrionUI.Theme.Accent,
        Key = options.Key or Enum.KeyCode.RightControl,
        Pages = {},
        IsOpen = true
    }

    -- Holder
    local ScreenGui = CreateInstance("ScreenGui", {
        Name = "OrionUI_" .. Window.Name,
        Parent = game:GetService("CoreGui"),
        ZIndexBehavior = Enum.ZIndexBehavior.Global,
        DisplayOrder = 9999,
        ResetOnSpawn = false
    })

    -- Main Container (for drag and visibility)
    local MainContainer = CreateInstance("Frame", {
        Parent = ScreenGui,
        BackgroundTransparency = 1,
        Position = UDim2.new(0.5, -330, 0.5, -280),
        Size = UDim2.new(0, 660, 0, 560),
        ClipsDescendants = true
    })

    -- Main Frame with border
    local MainFrameBorder = CreateInstance("Frame", {
        Parent = MainContainer,
        BackgroundColor3 = OrionUI.Theme.Outline,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 1, 0)
    })

    local MainFrame = CreateInstance("Frame", {
        Parent = MainFrameBorder,
        BackgroundColor3 = OrionUI.Theme.Background,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 1, 0, 1),
        Size = UDim2.new(1, -2, 1, -2)
    })

    -- Background Pattern
    local BGPattern = CreateInstance("ImageLabel", {
        Parent = MainFrame,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 1, 0),
        Image = "rbxassetid://15453092054",
        ImageColor3 = Color3.fromRGB(10, 10, 10),
        ImageTransparency = 0.7,
        ScaleType = Enum.ScaleType.Tile,
        TileSize = UDim2.new(0, 6, 0, 6),
        ZIndex = 0
    })

    -- Top Bar Background
    local TopGradient = CreateInstance("ImageLabel", {
        Parent = MainFrame,
        BackgroundTransparency = 1,
        Size = UDim2.new(1, 0, 0, 4),
        Image = "rbxassetid://15453122383",
        ImageColor3 = Window.Accent,
        ImageTransparency = 0.7,
        ScaleType = Enum.ScaleType.Slice,
        SliceCenter = Rect.new(4, 4, 4, 4),
        ZIndex = 2
    })

    -- Title Bar
    local TitleBar = CreateInstance("Frame", {
        Parent = MainFrame,
        BackgroundColor3 = OrionUI.Theme.Background2,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 40),
        ZIndex = 10
    })

    local Logo = CreateInstance("ImageLabel", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Size = UDim2.new(0, 24, 0, 24),
        Position = UDim2.new(0, 10, 0.5, -12),
        Image = Window.Logo,
        ImageColor3 = Window.Accent,
        ZIndex = 11
    })

    local TitleText = CreateInstance("TextLabel", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 42, 0, 6),
        Size = UDim2.new(1, -90, 0, 18),
        Text = Window.Name,
        FontFace = GetFont(),
        TextColor3 = OrionUI.Theme.Text,
        TextSize = 15,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 11
    })

    local SubTitleText = CreateInstance("TextLabel", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 42, 0, 20),
        Size = UDim2.new(1, -90, 0, 14),
        Text = Window.SubName,
        FontFace = GetFont(),
        TextColor3 = OrionUI.Theme.SubText,
        TextSize = 11,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 11
    })

    -- Close/Minimize Buttons
    local ButtonsContainer = CreateInstance("Frame", {
        Parent = TitleBar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -80, 0, 0),
        Size = UDim2.new(0, 70, 1, 0),
        ZIndex = 11
    })

    local CloseBtn = CreateInstance("TextButton", {
        Parent = ButtonsContainer,
        BackgroundColor3 = OrionUI.Theme.Danger,
        BorderSizePixel = 0,
        Position = UDim2.new(1, -35, 0.5, -8),
        Size = UDim2.new(0, 16, 0, 16),
        Text = "",
        AutoButtonColor = false,
        ZIndex = 12
    })
    Round(CloseBtn, 4)

    local MinimizeBtn = CreateInstance("TextButton", {
        Parent = ButtonsContainer,
        BackgroundColor3 = OrionUI.Theme.Element,
        BorderSizePixel = 0,
        Position = UDim2.new(1, -55, 0.5, -8),
        Size = UDim2.new(0, 16, 0, 16),
        Text = "",
        AutoButtonColor = false,
        ZIndex = 12
    })
    Round(MinimizeBtn, 4)

    local MinimizeLine = CreateInstance("Frame", {
        Parent = MinimizeBtn,
        BackgroundColor3 = OrionUI.Theme.SubText,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 3, 0.5, 0),
        Size = UDim2.new(0, 10, 0, 2),
        ZIndex = 13
    })

    local function HideWindow()
        Window.IsOpen = false
        MainContainer:TweenPosition(
            UDim2.new(0.5, 0, 0.5, 0),
            "In", "Quad", 0.2, true
        )
    end

    local function ShowWindow()
        Window.IsOpen = true
        MainContainer:TweenPosition(
            UDim2.new(0.5, -330, 0.5, -280),
            "Out", "Quad", 0.2, true
        )
    end

    CloseBtn.MouseButton1Click:Connect(HideWindow)
    MinimizeBtn.MouseButton1Click:Connect(function()
        if Window.IsOpen then
            HideWindow()
        else
            ShowWindow()
        end
    end)

    -- Hover effects
    CloseBtn.MouseEnter:Connect(function()
        CloseBtn.BackgroundColor3 = Color3.fromRGB(200, 40, 40)
    end)
    CloseBtn.MouseLeave:Connect(function()
        CloseBtn.BackgroundColor3 = OrionUI.Theme.Danger
    end)

    MinimizeBtn.MouseEnter:Connect(function()
        MinimizeBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    end)
    MinimizeBtn.MouseLeave:Connect(function()
        MinimizeBtn.BackgroundColor3 = OrionUI.Theme.Element
    end)

    -- Tabs Container
    local TabsContainer = CreateInstance("Frame", {
        Parent = MainFrame,
        BackgroundColor3 = OrionUI.Theme.Background2,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 0, 0, 40),
        Size = UDim2.new(0, 70, 1, -40),
        ZIndex = 5
    })

    local TabsList = CreateInstance("UIListLayout", {
        Parent = TabsContainer,
        Padding = UDim.new(0, 0),
        SortOrder = Enum.SortOrder.LayoutOrder
    })

    local TabsPadding = CreateInstance("UIPadding", {
        Parent = TabsContainer,
        PaddingTop = UDim.new(0, 6)
    })

    -- Top Side Fix
    local TopSideFix = CreateInstance("Frame", {
        Parent = TabsContainer,
        BackgroundColor3 = Color3.new(0, 0, 0),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 70, 0, 0),
        Size = UDim2.new(0, 1, 1, 0),
        ZIndex = 6
    })

    local TopSideFix2 = CreateInstance("Frame", {
        Parent = TopSideFix,
        BackgroundColor3 = Color3.fromRGB(40, 40, 40),
        BorderSizePixel = 0,
        Position = UDim2.new(1, 0, 0, 0),
        Size = UDim2.new(0, 1, 1, 0),
        ZIndex = 6
    })

    -- Pages Container
    local PagesContainer = CreateInstance("Frame", {
        Parent = MainFrame,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 74, 0, 44),
        Size = UDim2.new(1, -78, 1, -48),
        ZIndex = 3
    })

    -- Inner border for pages
    local PagesInnerBorder = CreateInstance("Frame", {
        Parent = PagesContainer,
        BackgroundColor3 = Color3.fromRGB(45, 45, 45),
        BorderSizePixel = 0,
        Position = UDim2.new(0, 1, 0, 0),
        Size = UDim2.new(1, -1, 1, 0),
        ZIndex = 3
    })

    local PagesInner = CreateInstance("Frame", {
        Parent = PagesInnerBorder,
        BackgroundColor3 = OrionUI.Theme.Background3,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 1, 0, 0),
        Size = UDim2.new(1, -1, 1, 0),
        ZIndex = 3
    })

    -- Make draggable (using title bar)
    AddDraggable(MainContainer, TitleBar)

    -- Window Functions
    function Window:SetOpen(state)
        Window.IsOpen = state
        MainContainer.Visible = state
    end

    function Window:Toggle()
        if Window.IsOpen then
            HideWindow()
        else
            ShowWindow()
        end
    end

    function Window:SetCenter()
        MainContainer.Position = UDim2.new(0.5, -MainContainer.AbsoluteSize.X/2, 0.5, -MainContainer.AbsoluteSize.Y/2)
    end

    function Window:GetVisible()
        return Window.IsOpen
    end

    -- Key toggle
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Window.Key then
            Window:Toggle()
        end
    end)

    -- Page Creation
    function Window:Page(options)
        options = options or {}
        local Page = {
            Name = options.Name or "Page",
            Icon = options.Icon or "rbxassetid://15453302474",
            Window = self,
            Sections = {}
        }

        -- Tab Button
        local TabBtn = CreateInstance("TextButton", {
            Parent = TabsContainer,
            BackgroundColor3 = OrionUI.Theme.Background2,
            BorderSizePixel = 0,
            Size = UDim2.new(0, 70, 0, 60),
            Text = "",
            AutoButtonColor = false,
            ZIndex = 6
        })

        -- Tab selection indicator (left bar)
        local TabIndicator = CreateInstance("Frame", {
            Parent = TabBtn,
            BackgroundColor3 = Window.Accent,
            BorderSizePixel = 0,
            Position = UDim2.new(0, 0, 0, 0),
            Size = UDim2.new(0, 3, 1, 0),
            Visible = false,
            ZIndex = 7
        })

        -- Tab hover indicator (top bar)
        local TabHover = CreateInstance("Frame", {
            Parent = TabBtn,
            BackgroundColor3 = Color3.new(0, 0, 0),
            BorderSizePixel = 0,
            Position = UDim2.new(0, 0, 0, -2),
            Size = UDim2.new(1, 0, 0, 1),
            Visible = false,
            ZIndex = 7
        })

        local TabIcon = CreateInstance("ImageLabel", {
            Parent = TabBtn,
            BackgroundTransparency = 1,
            Size = UDim2.new(0, 22, 0, 22),
            Position = UDim2.new(0.5, -11, 0, 12),
            Image = Page.Icon,
            ImageColor3 = OrionUI.Theme.SubText,
            ZIndex = 7
        })

        local TabText = CreateInstance("TextLabel", {
            Parent = TabBtn,
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 0, 0, 36),
            Size = UDim2.new(1, 0, 0, 14),
            Text = Page.Name,
            FontFace = GetFont(),
            TextColor3 = OrionUI.Theme.SubText,
            TextSize = 10,
            ZIndex = 7
        })

        -- Page Content
        local PageContent = CreateInstance("ScrollingFrame", {
            Parent = PagesInner,
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 1, 0),
            CanvasSize = UDim2.new(0, 0, 0, 0),
            ScrollBarThickness = 4,
            ScrollBarImageColor3 = Color3.fromRGB(60, 60, 60),
            ScrollBarImageTransparency = 0.5,
            Visible = false,
            ZIndex = 4
        })

        local PageList = CreateInstance("UIListLayout", {
            Parent = PageContent,
            Padding = UDim.new(0, 14),
            SortOrder = Enum.SortOrder.LayoutOrder,
            HorizontalAlignment = Enum.HorizontalAlignment.Center
        })

        local PagePadding = CreateInstance("UIPadding", {
            Parent = PageContent,
            PaddingTop = UDim.new(0, 12),
            PaddingLeft = UDim.new(0, 10),
            PaddingRight = UDim.new(0, 10)
        })

        PageList:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            PageContent.CanvasSize = UDim2.new(0, 0, 0, PageList.AbsoluteContentSize.Y + 20)
        end)

        -- Tab Logic
        local function Activate()
            for _, p in ipairs(Window.Pages) do
                p.Content.Visible = false
                p.TabBtn.BackgroundColor3 = OrionUI.Theme.Background2
                p.TabIndicator.Visible = false
                p.TabHover.Visible = false
                p.TabIcon.ImageColor3 = OrionUI.Theme.SubText
                p.TabText.TextColor3 = OrionUI.Theme.SubText
            end
            PageContent.Visible = true
            TabBtn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
            TabIndicator.Visible = true
            TabIcon.ImageColor3 = Window.Accent
            TabText.TextColor3 = Window.Accent
        end

        TabBtn.MouseButton1Click:Connect(Activate)

        TabBtn.MouseEnter:Connect(function()
            if not PageContent.Visible then
                TabBtn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
                TabHover.Visible = true
                TabIcon.ImageColor3 = Color3.fromRGB(180, 180, 180)
            end
        end)

        TabBtn.MouseLeave:Connect(function()
            if not PageContent.Visible then
                TabBtn.BackgroundColor3 = OrionUI.Theme.Background2
                TabHover.Visible = false
                TabIcon.ImageColor3 = OrionUI.Theme.SubText
            end
        end)

        -- Store page
        Page.Content = PageContent
        Page.TabBtn = TabBtn
        Page.TabIcon = TabIcon
        Page.TabText = TabText
        Page.TabIndicator = TabIndicator

        table.insert(Window.Pages, Page)

        if #Window.Pages == 1 then
            task.wait(0.1)
            Activate()
        end

        -- Section Creation
        function Page:Section(options)
            options = options or {}
            local Section = {
                Name = options.Name or "Section",
                Side = options.Side or "Left",
                Page = self,
                Window = Window
            }

            -- Determine position
            local sectionIndex = #self.Sections + 1
            local position = UDim2.new(0.5, -4, 0, 0)
            if Section.Side == "Right" then
                position = UDim2.new(0.5, 4, 0, 0)
            end

            local SectionFrame = CreateInstance("Frame", {
                Parent = PageContent,
                BackgroundColor3 = OrionUI.Theme.SectionBackground,
                BorderSizePixel = 0,
                Size = UDim2.new(0.5, -8, 0, 180),
                Position = position,
                ZIndex = 4
            })
            Round(SectionFrame, 6)

            -- Add glow to section
            local SectionGlow = AddGlow(SectionFrame, Window.Accent, 0.3)

            local SectionInner = CreateInstance("Frame", {
                Parent = SectionFrame,
                BackgroundColor3 = OrionUI.Theme.SectionInner,
                BorderSizePixel = 0,
                Position = UDim2.new(0, 1, 0, 1),
                Size = UDim2.new(1, -2, 1, -2)
            })
            Round(SectionInner, 5)

            -- Top bar
            local SectionTopBar = CreateInstance("Frame", {
                Parent = SectionInner,
                BackgroundColor3 = OrionUI.Theme.SectionTop,
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 0, 24),
                ZIndex = 5
            })

            local SectionTitle = CreateInstance("TextLabel", {
                Parent = SectionTopBar,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 0),
                Size = UDim2.new(1, -20, 1, 0),
                Text = Section.Name,
                FontFace = GetFont(),
                TextColor3 = OrionUI.Theme.SubText,
                TextSize = 11,
                TextXAlignment = Enum.TextXAlignment.Left,
                ZIndex = 6
            })

            local SectionContent = CreateInstance("Frame", {
                Parent = SectionInner,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 4, 0, 28),
                Size = UDim2.new(1, -8, 1, -32),
                ZIndex = 4
            })

            local SectionList = CreateInstance("UIListLayout", {
                Parent = SectionContent,
                Padding = UDim.new(0, 5),
                SortOrder = Enum.SortOrder.LayoutOrder
            })

            local SectionPadding = CreateInstance("UIPadding", {
                Parent = SectionContent,
                PaddingTop = UDim.new(0, 2),
                PaddingBottom = UDim.new(0, 4)
            })

            -- Toggle
            function Section:Toggle(options)
                options = options or {}
                local Toggle = {
                    Name = options.Name or "Toggle",
                    Flag = options.Flag,
                    Default = options.Default or false,
                    Callback = options.Callback or function() end,
                    State = false
                }
                Toggle.State = Toggle.Default

                local ToggleFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 26),
                    ZIndex = 5
                })

                local ToggleLabel = CreateInstance("TextLabel", {
                    Parent = ToggleFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -36, 1, 0),
                    Text = Toggle.Name,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local ToggleBox = CreateInstance("Frame", {
                    Parent = ToggleFrame,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Position = UDim2.new(1, 0, 0.5, 0),
                    Size = UDim2.new(0, 16, 0, 16),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    ZIndex = 6
                })
                Round(ToggleBox, 4)

                local ToggleBoxInner = CreateInstance("Frame", {
                    Parent = ToggleBox,
                    BackgroundColor3 = Window.Accent,
                    BorderSizePixel = 0,
                    Position = UDim2.new(0, 1, 0, 1),
                    Size = UDim2.new(1, -2, 1, -2),
                    ZIndex = 7
                })
                Round(ToggleBoxInner, 3)

                -- Add glow to toggle
                local ToggleGlow = AddGlow(ToggleBox, Window.Accent, 0.5)

                local ToggleBtn = CreateInstance("TextButton", {
                    Parent = ToggleFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 1, 0),
                    Text = "",
                    ZIndex = 10
                })

                local function UpdateToggle()
                    if Toggle.State then
                        ToggleBoxInner.Size = UDim2.new(1, -2, 1, -2)
                        ToggleLabel.TextColor3 = Window.Accent
                        ToggleGlow.ImageTransparency = 0.3
                    else
                        ToggleBoxInner.Size = UDim2.new(0, 0, 0, 0)
                        ToggleLabel.TextColor3 = OrionUI.Theme.Text
                        ToggleGlow.ImageTransparency = 1
                    end
                    Toggle.Callback(Toggle.State)
                    if Toggle.Flag then OrionUI.Flags[Toggle.Flag] = Toggle.State end
                end

                ToggleBtn.MouseButton1Click:Connect(function()
                    Toggle.State = not Toggle.State
                    UpdateToggle()
                end)

                function Toggle:Set(state)
                    Toggle.State = state
                    UpdateToggle()
                end

                function Toggle:Get()
                    return Toggle.State
                end

                if Toggle.Default then UpdateToggle() end

                return Toggle
            end

            -- Slider
            function Section:Slider(options)
                options = options or {}
                local Slider = {
                    Name = options.Name or "Slider",
                    Flag = options.Flag,
                    Min = options.Min or 0,
                    Max = options.Max or 100,
                    Default = options.Default or 50,
                    Suffix = options.Suffix or "",
                    Decimals = options.Decimals or 0,
                    Callback = options.Callback or function() end,
                    Value = 0
                }
                Slider.Value = Slider.Default

                local SliderFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 38),
                    ZIndex = 5
                })

                local SliderLabel = CreateInstance("TextLabel", {
                    Parent = SliderFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -50, 0, 16),
                    Text = Slider.Name,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local SliderValue = CreateInstance("TextLabel", {
                    Parent = SliderFrame,
                    AnchorPoint = Vector2.new(1, 0),
                    Position = UDim2.new(1, 0, 0, 0),
                    Size = UDim2.new(0, 45, 0, 16),
                    Text = tostring(Slider.Value) .. Slider.Suffix,
                    FontFace = GetFont(),
                    TextColor3 = Window.Accent,
                    TextSize = 11,
                    TextXAlignment = Enum.TextXAlignment.Right,
                    ZIndex = 6
                })

                local SliderTrack = CreateInstance("Frame", {
                    Parent = SliderFrame,
                    Position = UDim2.new(0, 0, 0, 22),
                    Size = UDim2.new(1, 0, 0, 8),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    ZIndex = 6
                })
                Round(SliderTrack, 4)

                local SliderFill = CreateInstance("Frame", {
                    Parent = SliderTrack,
                    BackgroundColor3 = Window.Accent,
                    BorderSizePixel = 0,
                    Size = UDim2.new(0.5, 0, 1, 0),
                    ZIndex = 7
                })
                Round(SliderFill, 4)

                -- Add glow to slider
                local SliderGlow = AddGlow(SliderFill, Window.Accent, 0.6)

                local SliderBtn = CreateInstance("TextButton", {
                    Parent = SliderFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 1, 0),
                    Text = "",
                    ZIndex = 10
                })

                local dragging = false

                local function UpdateSlider(input)
                    local x = math.clamp((input.Position.X - SliderTrack.AbsolutePosition.X) / SliderTrack.AbsoluteSize.X, 0, 1)
                    local val = math.round((Slider.Min + (Slider.Max - Slider.Min) * x) * (10 ^ Slider.Decimals)) / (10 ^ Slider.Decimals)
                    Slider.Value = val
                    SliderFill.Size = UDim2.new(x, 0, 1, 0)
                    SliderValue.Text = tostring(val) .. Slider.Suffix
                    
                    -- Update glow
                    SliderGlow.Size = UDim2.new(1, 20, 1, 20)
                    SliderGlow.Position = UDim2.new(0, -10, 0.5, -10)
                    
                    Slider.Callback(val)
                    if Slider.Flag then OrionUI.Flags[Slider.Flag] = val end
                end

                SliderBtn.InputBegan:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = true
                        UpdateSlider(input)
                    end
                end)

                SliderBtn.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging = false
                    end
                end)

                UserInputService.InputChanged:Connect(function(input)
                    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                        UpdateSlider(input)
                    end
                end)

                function Slider:Set(value)
                    Slider.Value = math.clamp(value, Slider.Min, Slider.Max)
                    local x = (Slider.Value - Slider.Min) / (Slider.Max - Slider.Min)
                    SliderFill.Size = UDim2.new(x, 0, 1, 0)
                    SliderValue.Text = tostring(Slider.Value) .. Slider.Suffix
                    Slider.Callback(Slider.Value)
                    if Slider.Flag then OrionUI.Flags[Slider.Flag] = Slider.Value end
                end

                function Slider:Get()
                    return Slider.Value
                end

                -- Set default
                local defaultX = (Slider.Default - Slider.Min) / (Slider.Max - Slider.Min)
                SliderFill.Size = UDim2.new(defaultX, 0, 1, 0)
                SliderValue.Text = tostring(Slider.Default) .. Slider.Suffix

                return Slider
            end

            -- Dropdown
            function Section:Dropdown(options)
                options = options or {}
                local Dropdown = {
                    Name = options.Name or "Dropdown",
                    Flag = options.Flag,
                    Items = options.Items or {"Option 1", "Option 2", "Option 3"},
                    Default = options.Default or 1,
                    Callback = options.Callback or function() end,
                    Value = "",
                    Index = 1
                }
                Dropdown.Index = Dropdown.Default
                Dropdown.Value = Dropdown.Items[Dropdown.Index]

                local DropdownFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 30),
                    ClipsDescendants = true,
                    ZIndex = 5
                })

                local DropdownLabel = CreateInstance("TextLabel", {
                    Parent = DropdownFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 14),
                    Text = Dropdown.Name,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local DropdownBtn = CreateInstance("TextButton", {
                    Parent = DropdownFrame,
                    Position = UDim2.new(0, 0, 0, 18),
                    Size = UDim2.new(1, 0, 0, 26),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    Text = "  " .. Dropdown.Value,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 11,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    AutoButtonColor = false,
                    ZIndex = 6
                })
                Round(DropdownBtn, 4)

                local DropdownIcon = CreateInstance("ImageLabel", {
                    Parent = DropdownBtn,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Position = UDim2.new(1, -8, 0.5, 0),
                    Size = UDim2.new(0, 10, 0, 10),
                    Image = "rbxassetid://6034818379",
                    ImageColor3 = OrionUI.Theme.SubText,
                    BackgroundTransparency = 1,
                    ZIndex = 7
                })

                local DropdownList = CreateInstance("Frame", {
                    Parent = DropdownFrame,
                    Position = UDim2.new(0, 0, 0, 46),
                    Size = UDim2.new(1, 0, 0, 0),
                    BackgroundColor3 = OrionUI.Theme.Background2,
                    BorderSizePixel = 0,
                    AutomaticSize = Enum.AutomaticSize.Y,
                    Visible = false,
                    ZIndex = 20
                })
                Round(DropdownList, 4)

                local DropdownListInner = CreateInstance("Frame", {
                    Parent = DropdownList,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 0),
                    AutomaticSize = Enum.AutomaticSize.Y,
                    ZIndex = 21
                })

                local DropdownListLayout = CreateInstance("UIListLayout", {
                    Parent = DropdownListInner,
                    Padding = UDim.new(0, 2),
                    SortOrder = Enum.SortOrder.LayoutOrder
                })

                local DropdownListPadding = CreateInstance("UIPadding", {
                    Parent = DropdownListInner,
                    PaddingTop = UDim.new(0, 4),
                    PaddingBottom = UDim.new(0, 4)
                })

                -- Create options
                for i, item in ipairs(Dropdown.Items) do
                    local OptionBtn = CreateInstance("TextButton", {
                        Parent = DropdownListInner,
                        BackgroundTransparency = 1,
                        Size = UDim2.new(1, 0, 0, 22),
                        Text = "  " .. item,
                        FontFace = GetFont(),
                        TextColor3 = OrionUI.Theme.SubText,
                        TextSize = 11,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        AutoButtonColor = false,
                        ZIndex = 22
                    })

                    OptionBtn.MouseButton1Click:Connect(function()
                        Dropdown.Index = i
                        Dropdown.Value = item
                        DropdownBtn.Text = "  " .. item
                        DropdownList.Visible = false
                        DropdownIcon:TweenRotation(0, "Out", "Quad", 0.2)
                        Dropdown.Callback(item)
                        if Dropdown.Flag then OrionUI.Flags[Dropdown.Flag] = item end
                    end)

                    OptionBtn.MouseEnter:Connect(function()
                        OptionBtn.TextColor3 = Window.Accent
                    end)

                    OptionBtn.MouseLeave:Connect(function()
                        OptionBtn.TextColor3 = OrionUI.Theme.SubText
                    end)
                end

                local isOpen = false

                DropdownBtn.MouseButton1Click:Connect(function()
                    isOpen = not isOpen
                    DropdownList.Visible = isOpen
                    if isOpen then
                        DropdownIcon:TweenRotation(180, "Out", "Quad", 0.2)
                    else
                        DropdownIcon:TweenRotation(0, "Out", "Quad", 0.2)
                    end
                end)

                function Dropdown:Set(value)
                    for i, item in ipairs(Dropdown.Items) do
                        if item == value then
                            Dropdown.Index = i
                            Dropdown.Value = item
                            DropdownBtn.Text = "  " .. item
                            Dropdown.Callback(item)
                            if Dropdown.Flag then OrionUI.Flags[Dropdown.Flag] = item end
                            break
                        end
                    end
                end

                function Dropdown:Get()
                    return Dropdown.Value
                end

                return Dropdown
            end

            -- Button
            function Section:Button(options)
                options = options or {}
                local Button = {
                    Name = options.Name or "Button",
                    Callback = options.Callback or function() end
                }

                local ButtonFrame = CreateInstance("TextButton", {
                    Parent = SectionContent,
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    Size = UDim2.new(1, 0, 0, 30),
                    Text = Button.Name,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    AutoButtonColor = false,
                    ZIndex = 5
                })
                Round(ButtonFrame, 4)

                -- Add glow to button
                local ButtonGlow = AddGlow(ButtonFrame, Window.Accent, 0.4)
                ButtonGlow.ImageTransparency = 1

                ButtonFrame.MouseButton1Click:Connect(function()
                    Button.Callback()
                end)

                ButtonFrame.MouseEnter:Connect(function()
                    ButtonFrame.BackgroundColor3 = Window.Accent
                    ButtonGlow.ImageTransparency = 0.4
                end)

                ButtonFrame.MouseLeave:Connect(function()
                    ButtonFrame.BackgroundColor3 = OrionUI.Theme.Element
                    ButtonGlow.ImageTransparency = 1
                end)

                return Button
            end

            -- Keybind
            function Section:Keybind(options)
                options = options or {}
                local Keybind = {
                    Name = options.Name or "Keybind",
                    Flag = options.Flag,
                    Default = options.Default or Enum.KeyCode.X,
                    Mode = options.Mode or "Toggle",
                    Callback = options.Callback or function() end,
                    Key = nil,
                    Active = false
                }
                Keybind.Key = Keybind.Default

                local KeybindFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 26),
                    ZIndex = 5
                })

                local KeybindLabel = CreateInstance("TextLabel", {
                    Parent = KeybindFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -70, 1, 0),
                    Text = Keybind.Name,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local KeybindValue = CreateInstance("TextLabel", {
                    Parent = KeybindFrame,
                    AnchorPoint = Vector2.new(1, 0),
                    Position = UDim2.new(1, 0, 0, 0),
                    Size = UDim2.new(0, 65, 1, 0),
                    Text = "[" .. string.sub(tostring(Keybind.Key.Name), -4) .. "]",
                    FontFace = GetFont(),
                    TextColor3 = Window.Accent,
                    TextSize = 10,
                    TextXAlignment = Enum.TextXAlignment.Right,
                    ZIndex = 6
                })

                local KeybindBtn = CreateInstance("TextButton", {
                    Parent = KeybindFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 1, 0),
                    Text = "",
                    ZIndex = 10
                })

                local listening = false

                KeybindBtn.MouseButton1Click:Connect(function()
                    listening = true
                    KeybindValue.Text = "[...]"
                    KeybindValue.TextColor3 = OrionUI.Theme.Danger
                end)

                UserInputService.InputBegan:Connect(function(input)
                    if listening then
                        if input.UserInputType == Enum.UserInputType.Keyboard then
                            Keybind.Key = input.KeyCode
                            KeybindValue.Text = "[" .. string.sub(tostring(input.KeyCode.Name), -4) .. "]"
                            KeybindValue.TextColor3 = Window.Accent
                            listening = false
                            if Keybind.Flag then OrionUI.Flags[Keybind.Flag] = input.KeyCode end
                        end
                    else
                        if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Keybind.Key then
                            if Keybind.Mode == "Toggle" then
                                Keybind.Active = not Keybind.Active
                                Keybind.Callback(Keybind.Active)
                            elseif Keybind.Mode == "Hold" then
                                Keybind.Active = true
                                Keybind.Callback(true)
                            end
                            if Keybind.Flag then OrionUI.Flags[Keybind.Flag] = {Key = Keybind.Key, Active = Keybind.Active} end
                        end
                    end
                end)

                UserInputService.InputEnded:Connect(function(input)
                    if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Keybind.Key and Keybind.Mode == "Hold" then
                        Keybind.Active = false
                        Keybind.Callback(false)
                        if Keybind.Flag then OrionUI.Flags[Keybind.Flag] = {Key = Keybind.Key, Active = false} end
                    end
                end)

                function Keybind:Set(key)
                    Keybind.Key = key
                    KeybindValue.Text = "[" .. string.sub(tostring(key.Name), -4) .. "]"
                end

                function Keybind:Get()
                    return {Key = Keybind.Key, Mode = Keybind.Mode, Active = Keybind.Active}
                end

                return Keybind
            end

            -- Colorpicker
            function Section:Colorpicker(options)
                options = options or {}
                local Colorpicker = {
                    Name = options.Name or "Colorpicker",
                    Flag = options.Flag,
                    Default = options.Default or Color3.fromRGB(255, 255, 255),
                    Callback = options.Callback or function() end,
                    Color = nil,
                    Alpha = 0
                }
                Colorpicker.Color = Colorpicker.Default

                local ColorpickerFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 26),
                    ZIndex = 5
                })

                local ColorpickerLabel = CreateInstance("TextLabel", {
                    Parent = ColorpickerFrame,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, -36, 1, 0),
                    Text = Colorpicker.Name,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 12,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })

                local ColorpickerPreview = CreateInstance("Frame", {
                    Parent = ColorpickerFrame,
                    AnchorPoint = Vector2.new(1, 0.5),
                    Position = UDim2.new(1, 0, 0.5, 0),
                    Size = UDim2.new(0, 24, 0, 16),
                    BackgroundColor3 = Colorpicker.Color,
                    BorderSizePixel = 0,
                    ZIndex = 6
                })
                Round(ColorpickerPreview, 3)

                function Colorpicker:Set(color, alpha)
                    Colorpicker.Color = color
                    Colorpicker.Alpha = alpha or 0
                    ColorpickerPreview.BackgroundColor3 = color
                    Colorpicker.Callback(color, Colorpicker.Alpha)
                    if Colorpicker.Flag then OrionUI.Flags[Colorpicker.Flag] = {Color = color, Alpha = Colorpicker.Alpha} end
                end

                function Colorpicker:Get()
                    return Colorpicker.Color, Colorpicker.Alpha
                end

                return Colorpicker
            end

            -- Label
            function Section:Label(text)
                local LabelFrame = CreateInstance("TextLabel", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 18),
                    Text = text,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.SubText,
                    TextSize = 11,
                    TextWrapped = true,
                    ZIndex = 5
                })

                function LabelFrame:SetText(text)
                    LabelFrame.Text = text
                end

                return LabelFrame
            end

            -- Textbox
            function Section:Textbox(options)
                options = options or {}
                local Textbox = {
                    Name = options.Name or "Textbox",
                    Flag = options.Flag,
                    Default = options.Default or "",
                    Callback = options.Callback or function() end,
                    Value = ""
                }
                Textbox.Value = Textbox.Default

                local TextboxFrame = CreateInstance("Frame", {
                    Parent = SectionContent,
                    BackgroundTransparency = 1,
                    Size = UDim2.new(1, 0, 0, 30),
                    ZIndex = 5
                })

                local TextboxInput = CreateInstance("TextBox", {
                    Parent = TextboxFrame,
                    Position = UDim2.new(0, 0, 0, 8),
                    Size = UDim2.new(1, 0, 0, 22),
                    BackgroundColor3 = OrionUI.Theme.Element,
                    BorderSizePixel = 0,
                    Text = Textbox.Default,
                    PlaceholderText = Textbox.Name,
                    PlaceholderColor3 = OrionUI.Theme.SubText,
                    FontFace = GetFont(),
                    TextColor3 = OrionUI.Theme.Text,
                    TextSize = 11,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    ZIndex = 6
                })
                Round(TextboxInput, 4)

                local TextboxPadding = CreateInstance("UIPadding", {
                    Parent = TextboxInput,
                    PaddingLeft = UDim.new(0, 8)
                })

                TextboxInput.FocusLost:Connect(function()
                    Textbox.Value = TextboxInput.Text
                    Textbox.Callback(Textbox.Value)
                    if Textbox.Flag then OrionUI.Flags[Textbox.Flag] = Textbox.Value end
                end)

                function Textbox:Set(value)
                    Textbox.Value = value
                    TextboxInput.Text = value
                end

                function Textbox:Get()
                    return Textbox.Value
                end

                return Textbox
            end

            return Section
        end

        return Page
    end

    -- Notification
    function OrionUI:Notification(options)
        options = options or {}
        local Notification = {
            Title = options.Title or "Notification",
            Description = options.Description or "",
            Duration = options.Duration or 3,
            Icon = options.Icon
        }

        local NotifFrame = CreateInstance("Frame", {
            Parent = ScreenGui,
            BackgroundColor3 = OrionUI.Theme.Background2,
            BorderSizePixel = 0,
            Position = UDim2.new(1, -220, 0, 20),
            Size = UDim2.new(0, 200, 0, 50),
            AutomaticSize = Enum.AutomaticSize.Y,
            ClipsDescendants = true,
            ZIndex = 100
        })
        Round(NotifFrame, 8)

        local NotifPadding = CreateInstance("UIPadding", {
            Parent = NotifFrame,
            PaddingAll = UDim.new(0, 12)
        })

        local NotifTitle = CreateInstance("TextLabel", {
            Parent = NotifFrame,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 16),
            Text = Notification.Title,
            FontFace = GetFont(),
            TextColor3 = Window.Accent,
            TextSize = 13,
            TextXAlignment = Enum.TextXAlignment.Left,
            ZIndex = 101
        })

        local NotifDesc = CreateInstance("TextLabel", {
            Parent = NotifFrame,
            BackgroundTransparency = 1,
            Size = UDim2.new(1, 0, 0, 14),
            Position = UDim2.new(0, 0, 0, 18),
            Text = Notification.Description,
            FontFace = GetFont(),
            TextColor3 = OrionUI.Theme.SubText,
            TextSize = 11,
            TextWrapped = true,
            ZIndex = 101
        })

        -- Animation
        NotifFrame.Position = UDim2.new(1, 0, 0, 20)
        NotifFrame:TweenPosition(UDim2.new(1, -220, 0, 20), "Out", "Quad", 0.3)

        task.delay(Notification.Duration, function()
            NotifFrame:TweenPosition(UDim2.new(1, 220, 0, 20), "In", "Quad", 0.3, true)
            task.wait(0.3)
            if NotifFrame and NotifFrame.Parent then
                NotifFrame:Destroy()
            end
        end)
    end

    -- Config
    function OrionUI:GetConfig()
        return HttpService:JSONEncode(OrionUI.Flags)
    end

    function OrionUI:LoadConfig(config)
        local success, decoded = pcall(function()
            return HttpService:JSONDecode(config)
        end)
        if success then
            for k, v in pairs(decoded) do
                OrionUI.Flags[k] = v
            end
        end
    end

    -- Theme settings
    function OrionUI:SetGlow(enabled, intensity)
        OrionUI.Theme.GlowEnabled = enabled
        OrionUI.Theme.GlowIntensity = intensity or 0.6
    end

    function OrionUI:SetAccent(color)
        OrionUI.Theme.Accent = color
        Window.Accent = color
    end

    return Window
end

return OrionUI
