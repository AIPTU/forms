# forms

![build](https://img.shields.io/github/actions/workflow/status/Frago9876543210/forms/main.yml)
![php](https://img.shields.io/badge/php-8.1-informational)
![api](https://img.shields.io/badge/pocketmine-5.0-informational)

Modern version of PocketMine forms API, ported to PHP 8.1+ with high quality code and PHPStan integration.

Unlike other libraries, this one was created based on real problems in plugins. Therefore, you can see here rich API for
solving wide variety of problems.

## Table of contents

+ [Installation](#installation)
  - [As plugin](#as-plugin)
  - [As virion](#as-virion)
+ [ModalForm](#modalform)
  - [Using ModalForm to represent "yes" / "no" button clicks as `bool` in closure](#using-modalform-to-represent-yes--no-button-clicks-as-bool-in-closure)
  - [Short version of ModalForm to confirm any action](#short-version-of-modalform-to-confirm-any-action)
+ [MenuForm](#menuform)
  - [Using MenuForm to display buttons with icons from URL and path](#using-menuform-to-display-buttons-with-icons-from-url-and-path)
  - [Creating MenuForm from array of strings (i.e. from `string[]`) with modern syntax of matching button clicks](#creating-menuform-from-array-of-strings-ie-from-string-with-modern-syntax-of-matching-button-clicks)
  - [Appending MenuForm with new options to handle different permissions](#appending-menuform-with-new-options-to-handle-different-permissions)
+ [CustomForm](#customform)
  - [Using CustomForm with strict-typed API](#using-customform-with-strict-typed-api)
  - [Using CustomForm with less strict-typed API](#using-customform-with-less-strict-typed-api)

### Installation

> [!TIP]
> You can check out the [demo plugin](https://github.com/Frago9876543210/forms-demo) which shows how to use its API in
> plugin.

#### As plugin

This library can be loaded as plugin phar. You can use
the [`depend`](https://doc.pmmp.io/en/rtfd/developer-reference/plugin-manifest.html#depend) key in `plugin.yml` to
require its presence.

```yaml
depend:
  - forms
```

#### As virion

This library supports being included as [virion](https://github.com/poggit/support/blob/master/virion.md).

If you use [Poggit](https://poggit.pmmp.io) to build your plugin, you can add it to your `.poggit.yml` like so:

```yml
projects:
  YourPlugin:
    libs:
      - src: Frago9876543210/forms/libforms
        version: ^1.0.0
```

### ModalForm

#### Using ModalForm to represent "yes" / "no" button clicks as `bool` in closure

```php
$player->sendForm(new ModalForm("A small question", "Is our server cool?",
	//result of pressing the "yes" / "no" button is written to variable $choice
	function(Player $player, bool $choice) : void{
		$player->sendMessage($choice ? "Thank you" : "We will try to become better");
	}
));
```

![modal1](https://i.imgur.com/4Hf8RgD.png)

#### Short version of ModalForm to confirm any action

```php
$player->sendForm(ModalForm::confirm("Teleport request", "Do you want to accept it?",
	//called only when the player selects the "yes" button
	function(Player $player) : void{
		$player->sendMessage("*teleporting*");
	}
));
```

![modal2](https://i.imgur.com/jevWyHy.png)

### MenuForm

#### Using MenuForm to display buttons with icons from URL and path

> [!NOTE]
> If you are having trouble loading images, try installing https://github.com/Muqsit/FormImagesFix

```php
$player->sendForm(new MenuForm("Select server", "Choose server", [
	//buttons without icon
	new Button("SkyWars #1"),
	new Button("SkyWars #2"),
	//URL and path are supported for image
	new Button("SkyWars #3", Image::url("https://static.wikia.nocookie.net/minecraft_gamepedia/images/f/f0/Melon_JE2_BE2.png")),
	new Button("SkyWars #4", Image::path("textures/items/apple.png")),
], function(Player $player, Button $selected) : void{
	$player->sendMessage("You selected: " . $selected->text);
	$player->sendMessage("Index of button: " . $selected->getValue());
}));
```

![menu1](https://i.imgur.com/304JS1n.png)

#### Creating MenuForm from array of strings (i.e. from `string[]`) with modern syntax of matching button clicks

```php
//MenuForm::withOptions is useful when you have string[]
//syntax TIP: fn() added since PHP 7.4, match since PHP 8.0
$player->sendForm(MenuForm::withOptions("Select option", "List of options:", [
	"opt1", "opt2", "opt3",
	"default branch #1",
	"default branch #2",
], fn(Player $player, Button $selected) => match($selected->getValue()){
	0 => $player->sendMessage("message #1"), //opt1
	1 => $player->sendMessage("message #2"), //opt2
	2 => $player->sendMessage("message #3"), //opt3
	default => $player->sendMessage("You selected: " . $selected->text),
}));
```

![menu2](https://i.imgur.com/JXRfoJW.png)

#### Appending MenuForm with new options to handle different permissions

```php
//appending form data if Player has enough permissions
$form = MenuForm::withOptions("Player info", "Username: " . $player->getName(), [
	"view statistics", //accessible for all
], fn(Player $player, Button $selected) => match($selected->getValue()){
	0 => $player->sendMessage("*statistics*"),
	1 => $player->kick("kick message"),
	2 => $player->sendMessage("*logs*"),
	default => throw new \AssertionError("unreachable code"), //shut PHPStan
});

$isOp = $player->hasPermission(DefaultPermissions::ROOT_OPERATOR); //since PM 4.0
if($isOp){ //accessible for ops
	$form->appendOptions("kick player", "view logs");
}
$player->sendForm($form);
```

![menu3](https://i.imgur.com/5XTOe9d.png)

### CustomForm

#### Using CustomForm with strict-typed API

```php
$player->sendForm(new CustomForm("Enter data", [
	new Dropdown("Select product", ["beer", "cheese", "cola"]),
	new Input("Enter your name", "Bob"),
	new Label("I am label!"), //Note: get<BaseElement>() does not work with label
	new Slider("Select count", 0.0, 100.0, 1.0, 50.0),
	new StepSlider("Select product", ["beer", "cheese", "cola"]),
	new Toggle("Creative", $player->isCreative()),
], function(Player $player, CustomFormResponse $response) : void{
	$dropdown = $response->getDropdown();
	$player->sendMessage("You selected: " . $dropdown->getSelectedOption());

	$input = $response->getInput();
	$player->sendMessage("Your name is " . $input->getValue());

	$slider = $response->getSlider();
	$player->sendMessage("Count: " . $slider->getValue());
	//Note: `$slider->getValue()` can be of type `int|float`, so use type casting

	$stepSlider = $response->getStepSlider();
	$player->sendMessage("You selected: " . $stepSlider->getSelectedOption());

	$toggle = $response->getToggle();
	$player->setGamemode($toggle->getValue() ? GameMode::CREATIVE : GameMode::SURVIVAL);
}));
```

#### Using CustomForm with less strict-typed API

```php
$player->sendForm(new CustomForm("Enter data", [
	new Dropdown("Select product", ["beer", "cheese", "cola"]),
	new Input("Enter your name", "Bob"),
	new Label("I am label!"), //Note: get<BaseElement>() does not work with label
	new Slider("Select count", 0.0, 100.0, 1.0, 50.0),
	new StepSlider("Select product", ["beer", "cheese", "cola"]),
	new Toggle("Creative", $player->isCreative()),
], function(Player $player, CustomFormResponse $response) : void{
	/**
	 * type-hints for PHPStan
	 * @var string $product1
	 * @var string $username
	 * @var int|float $count
	 * @var string $product2
	 * @var bool $enableCreative
	 */
	[$product1, $username, $count, $product2, $enableCreative] = $response->getValues();

	//Note: `$count` can be of type `int|float`, so use type casting

	$player->sendMessage("You selected: $product1");
	$player->sendMessage("Your name is $username");
	$player->sendMessage("Count: $count");
	$player->sendMessage("You selected: $product2");
	$player->setGamemode($enableCreative ? GameMode::CREATIVE : GameMode::SURVIVAL);
}));
```

![custom1](https://i.imgur.com/BEIZdvO.png)
![custom2](https://i.imgur.com/AtRvSjp.png)
