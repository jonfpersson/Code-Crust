---
layout: post
title:  "Tamagotchi game development in C++"
date:   2023-02-10 19:30:00 +0100
author: Jonathan Persson
categories: Game Development
---

I've always been fascinated by Tamagotchis as a kid, and although I never had one for myself, I've always dreamed of having one. 
That's why I decided to make one from scratch but with a twist. Instead of some generic monster, I decided to go with one of my favorite Digimon, Agumon.

To give myself a challenge, I decided to write the project in C++ as I had prior experience with the C language. Knowing the graphics would be a pain to code myself, I scoured the internet for a graphics library and ended up with `SFML`.

`SFML` is a portable and easy-to-use media library for C++. It provides a simple interface to various components of your PC. In my project, I mainly used it for displaying different kinds of graphics, like sprites and windows.

# Graphics

Now, you can't have a game without some textures, so I had to make some myself!
I started up my favorite open-source illustration program, gimp, and began to create some pixel art.

<img src="{{ site.baseurl }}/assets/2023-02-01/koromon_atlas.png">
In this case, you'll see the picture consisting of two frames, firstly these are meant for creating a simple animation and secondly, having them combined into one image will reduce our IO-operations as they'll always be loaded into memory.

Then I just did the same thing for all of the Digimon evolutions I wanted, here are some examples.
<img src="{{ site.baseurl }}/assets/2023-02-01/0_botamon_texture_atlas">
<img src="{{ site.baseurl }}/assets/2023-02-01/2_agumon_texture_atlas_2">

# Code structure

Seeing as this was my first private C++ project, the structure underwent various iterations. There always seemed to be a better way of doing things. Firstly, I made two classes to represent the fundamental components of my game, a monster class and a UI controller.

{% highlight cpp %}

#include <SFML/Graphics.hpp>
#include "vitalDisplay.h"
class UIcontroller
{
private:
	void initTextField(sf::Text* const, int, int);
	
	sf::Font  m_font;
	sf::RenderWindow* window;
	vitalDisplay* vitals;
	const char* icons[3] = {"icons\\pizza.png", "icons\\heart.png", "icons\\water.png"};
	int m_size = 3;

public:
	~UIcontroller();
	UIcontroller(sf::RenderWindow* const);
	void notifyOfChange(int, int, int);
	void draw();

	static void setSprite(const char* const,
                          sf::Texture* const,
		                  const sf::IntRect* const,
		                  sf::Sprite* const, 
		                  int, 
		                  int, 
		                  int);
};

{% endhighlight %}

### UIcontroller.h

{% highlight cpp %}

#include <SFML/Graphics.hpp>
#include  "UIcontroller.h"

#define VITAL_START_VALUE 100

class Monster
{
private:
	std::vector<std::string> m_textures;
	int   m_x;
	int   m_y;
	int m_pixelScale = 5;
	int m_movementSpeedX = 5;
	int m_movementSpeedY = 3;
	long m_timer = 0;
	bool isGameOver = false;

	int m_food = VITAL_START_VALUE;
	int m_hp = VITAL_START_VALUE;
	int m_water = VITAL_START_VALUE;
	
	UIcontroller* m_uiController;

	sf::Texture m_texture;
	sf::IntRect m_rectagleSource;
	sf::Sprite  m_sprite;

	sf::RenderWindow* window;
	void updateVitals();
	void nextAtlasSquare();
	int currentAtlasSquare();
	void changeSpeedDirection(int* const);
	void handleEdgeColission(int, int);
	void setSprite(const char* const);
	void updateAtlas();
	void flipSprite();
	void move(int, int);
	void setIntRect(int, int, int, int);

public: 
	Monster(int x, int y, sf::IntRect rs, sf::RenderWindow* const);
	~Monster();

	sf::Sprite  getSprite();

	void draw();
	void animate(sf::Clock* const, int, int);
	void run(sf::Clock* const, int, int);
	void giveVitalPoint(int, int, int);
};

{% endhighlight %}

### Monster.h

# Conclusion
Overall this was a fun project, I learned a lot about C++ and using a media library. Even though `SFML` worked great for creating graphics I'd strongly recommend just using a game engine like Unity. An engine takes care of the less fun things leaving you the time and will to live to continue developing games.

<img src="{{ site.baseurl }}/assets/2023-02-01/final.png">

Want to read more of my code? Check out the project on my [Github][github].

[github]: https://www.github.com/jonfpersson
