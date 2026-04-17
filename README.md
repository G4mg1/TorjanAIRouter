-- i made this by using notepad so it could sometimes go wrong becuase i didnt debugged it 
local OpenRouter = {}
OpenRouter, limit = OpenRouter, {}

local maxlimit = 2000
local baseUrl = "https://router.huggingface.co/v1"
local HttpService = game:GetService("HttpService")
local hfToken = getgenv().API

if getgenv().IsLoaded then
  print("Sorry, cannot create multiple requests. Script needs to be run once and called using hooks.")
  return
else
  getgenv().IsLoaded = true
end

local httpquest = syn.request or http_request or request

function OpenRouter.newAI(AIModel, HK_API)
  getgenv().API = HK_API
  hfToken = HK_API

  local new = {}
  setmetatable(new, { __index = OpenRouter })

  function new:Request(request)
    local payload = {
      model = AIModel or "moonshotai/Kimi-K2-Instruct-0905",
      messages = {
        { role = "user", content = request }
      }
    }

    local body = HttpService:JSONEncode(payload)

    local data = {
      Url = baseUrl .. "/chat/completions",
      Method = "POST",
      Headers = {
        ["Content-Type"] = "application/json",
        ["Authorization"] = "Bearer " .. hfToken
      },
      Body = body
    }

    local response
    local success, errorMessage = pcall(function()
      response = httpquest(data)
    end)

    if success and response then
      if response.StatusCode == 200 then
        local responseData = HttpService:JSONDecode(response.Body)
        return responseData.choices[1].message.content
      else
        warn("Request failed with status: " .. tostring(response.StatusCode))
      end
    else
      warn("Error: " .. tostring(errorMessage))
    end
  end

  function new:SetToken(newToken)
    hfToken = newToken
    print("Token updated.")
  end

  function new:SetModel(newModel)
    AIModel = newModel
    print("Model updated to: " .. newModel)
  end

  function new:GetModelInfo()
    return { model = AIModel, token = hfToken }
  end

  function new:ResetLimit()
    limit[AIModel] = 0
    print("Limit reset.")
  end

  function new:GetLimit()
    return limit[AIModel] or 0
  end

  return new
end
