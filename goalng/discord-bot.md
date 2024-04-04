# Discord Bot

บงความนี้จะมาลองเขียน bot ของ discord ด้วย goalng  กันเล่นๆดู

## Project And Structure

เริ่มต้นก็มาสร้าง project กันก่อน

```
mkdir discord-bot-basic
cd discord-bot-basic
```

ต่อไปก็ set  go module

```
go mod init github.com/MumAroi/discord-bot-basic
```

หลังจากนั้นก็มาทำการ set structure project ของเรากันต่อตามนี้

```
golang-mysql-api
├── app
├── config
├── handlers
├── libraries
├── routes
```

## Libraries

ขั้นแรกมาสร้าง โครงสร้าง ของตัว libraries กันก่อน

ภายใน libraries จะประกอบไปด้วย&#x20;

* command.go
* contex.go
* handler.go

```
golang-mysql-api
├── app
├── config
├── handlers
├── libraries
│    ├── command.go
│    ├── contex.go
│    ├── handler.go
├── routes
```

command.go

```
package libraries

type (
	Command func(Context)

	CommandStruct struct {
		command Command
		help    string
	}

	CmdMap map[string]CommandStruct

	CommandHandler struct {
		cmds CmdMap
	}
)

func NewCommandHandler() *CommandHandler {
	return &CommandHandler{make(CmdMap)}
}

func (handler CommandHandler) GetCmds() CmdMap {
	return handler.cmds
}

func (handler CommandHandler) Get(name string) (*Command, bool) {
	cmd, found := handler.cmds[name]
	return &cmd.command, found
}

func (handler CommandHandler) Register(name string, command Command, helpmsg string) {
	cmdstruct := CommandStruct{command: command, help: helpmsg}
	handler.cmds[name] = cmdstruct
	if len(name) > 1 {
		handler.cmds[name[:1]] = cmdstruct
	}
}

func (command CommandStruct) GetHelp() string {
	return command.help
}
```

contex.go

```
package libraries

import (
	"fmt"

	"github.com/bwmarrin/discordgo"
)

type Context struct {
	Discord     *discordgo.Session
	Guild       *discordgo.Guild
	TextChannel *discordgo.Channel
	User        *discordgo.User
	Message     *discordgo.MessageCreate
	Args        []string
}

func NewContext(discord *discordgo.Session, guild *discordgo.Guild, textChannel *discordgo.Channel,
	user *discordgo.User, message *discordgo.MessageCreate, cmdHandler *CommandHandler,
) *Context {
	ctx := new(Context)
	ctx.Discord = discord
	ctx.Guild = guild
	ctx.TextChannel = textChannel
	ctx.User = user
	return ctx
}

func (ctx Context) Reply(content string) *discordgo.Message {
	msg, err := ctx.Discord.ChannelMessageSend(ctx.TextChannel.ID, content)
	if err != nil {
		fmt.Println("Error whilst sending message,", err)
		return nil
	}
	return msg
}
```

handler.go

```
package libraries

import (
	"fmt"
	"strings"

	"github.com/bwmarrin/discordgo"
)

type Handler struct {
	Prefix string
	BotId  string
	CMDH   *CommandHandler
}

func NewHandler(prefix string, botId string, cmdh *CommandHandler) *Handler {
	handler := new(Handler)
	handler.Prefix = prefix
	handler.BotId = botId
	handler.CMDH = cmdh
	return handler
}

func (h Handler) GetHandlers(discord *discordgo.Session, message *discordgo.MessageCreate) {
	user := message.Author
	if user.ID == h.BotId || user.Bot {
		return
	}

	content := message.Content
	if len(content) <= len(h.Prefix) {
		return
	}

	if content[:len(h.Prefix)] != h.Prefix {
		return
	}
	content = content[len(h.Prefix):]
	if len(content) < 1 {
		return
	}

	args := strings.Fields(content)
	name := strings.ToLower(args[0])
	command, found := h.CMDH.Get(name)
	if !found {
		return
	}

	channel, err := discord.State.Channel(message.ChannelID)
	if err != nil {
		fmt.Println("Error getting channel,", err)
		return
	}
	guild, err := discord.State.Guild(channel.GuildID)
	if err != nil {
		fmt.Println("Error getting guild,", err)
		return
	}
	ctx := NewContext(discord, guild, channel, user, message, h.CMDH)
	ctx.Args = args[1:]
	c := *command
	c(*ctx)
}
```

## Handlers

ถัดมาก็ทำการ สร้าง handler file เพี่อมาจัดการ logic ของคำสั่งต่างๆ

ในที่นี้จะสร้างเป็น pingpong.go&#x20;

```
golang-mysql-api
├── app
├── config
├── handlers
│    ├── pingpong.go
├── libraries
├── routes
```

pingpong.go code

```
package handlers

import (
	"fmt"

	"github.com/MumAroi/discord-bot-basic/libraries"
)

func PingPong(ctx libraries.Context) {

	_, err := ctx.Discord.ChannelMessageSend(ctx.TextChannel.ID, "Pong!")
	if err != nil {
		fmt.Println("Error whilst sending message,", err)
		return
	}

}
```

## Routes

ต่อมาก็มาสร้าง route ซึ่งเป็นตัวที่ใช้ในการจัดการ เส้นทางต่างๆของ command bot

```
golang-mysql-api
├── app
├── config
├── handlers
├── libraries
├── routes
│    ├── command.go
```

command.go code

```
package routes

import (
	"github.com/MumAroi/discord-bot-basic/handlers"
	"github.com/MumAroi/discord-bot-basic/libraries"
)

func RegisterCommands() *libraries.CommandHandler {
	commands := libraries.NewCommandHandler()
	commands.Register("pingpong", handlers.PingPong, "Ping Pong!")
	return commands
}
```

## Connecting bot

สุดท้ายแล้วมา สร้าง file app.go เพื่อสั่งให้ bot ทำงานกัน

```
golang-mysql-api
├── app
│    ├── app.go
├── config
├── handlers
├── libraries
├── routes
```

app.go code

```
package app

import (
	"fmt"
	"os"
	"os/signal"
	"syscall"

	"github.com/MumAroi/discord-bot-basic/config"
	"github.com/MumAroi/discord-bot-basic/libraries"
	"github.com/MumAroi/discord-bot-basic/routes"
	"github.com/bwmarrin/discordgo"
)

var (
	BOTTOKEN string
	PREFIX   string
)

func init() {
	_config := config.LoadConfig()
	BOTTOKEN = _config.BotToken
	PREFIX = _config.Prefix
}

func Run() {

	dg, err := discordgo.New("Bot " + BOTTOKEN)

	if err != nil {
		fmt.Println("error creating Discord session,", err)
		return
	}

	usr, err := dg.User("@me")
	if err != nil {
		fmt.Println("Error obtaining account details,", err)
		return
	}

	botId := usr.ID

	commands := routes.RegisterCommands()
	commandHandler := libraries.NewHandler(PREFIX, botId, commands)

	dg.AddHandler(commandHandler.GetHandlers)

	err = dg.Open()

	if err != nil {
		fmt.Println("error opening connection,", err)
		return
	}

	fmt.Println("Bot is now running. Press CTRL-C to exit.")

	sc := make(chan os.Signal, 1)

	signal.Notify(sc, syscall.SIGINT, syscall.SIGTERM, os.Interrupt, os.Kill)

	<-sc

	dg.Close()
}
```

หลังจากนั้น สร้าง file main.go เพื่อ เป็น file หลังในการสั่งเริ่มการทำการ

```
golang-mysql-api
├── app
├── config
├── handlers
├── libraries
├── routes
├── main.go
```

main.go code

```
package main

import (
	"github.com/MumAroi/discord-bot-basic/app"
)

func main() {
	app.Run()
}
```

แล้วก็อย่าลืมสร้าง file .env  เพื่อ set token ที่ได้จาก discord app bot

.env code

```
DISCORD_TOKEN=""
PREFIX=!
```

> final project: [https://github.com/MumAroi/discord-bot-basic](https://github.com/MumAroi/discord-bot-basic)

จากนั้นก็ลอง run bot กันเลย

```
go run main.go
```

The end.
