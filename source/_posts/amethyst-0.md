---
title: Amethyst引擎初见
date: 2019-07-25 08:59:30
tags: [rust, engine, amethyst]
---

这是个啥
======

[Amethyst](https://amethyst.rs/)是一个游戏引擎。

嗯？又一个游戏引擎？这个世界上最不缺的就是游戏引擎了，到网上随便一搜，你会发现随便一个阿猫阿狗都在做引擎（当然他们都是厉害的阿猫阿狗），那为什么要专门提一下Amethyst呢？

游戏引擎的选择需要考虑很多因素，包括价格，语言，性能，官方支持情况，易用性，等等。不同的需求会产生不同的选择。
我只说一下我为什么会关注Amethyst。

## rust

核心的原因在于它是用rust写成，并且能够使用rust直接调用的。

[rust](https://www.rust-lang.org/)是一门高效，安全而且很有生产力的语言。它还比较新，但是已经表现出惊人的实力，正在迅速渗透到各个领域。

我在18年开始关注rust并开始学习，为了能更好更深入地掌握这门技术，我决定尽量把所有的事情都用rust来做，放弃手上在用的其他语言。

## 游戏

另一方面，18年我关注了一个游戏引擎叫[godot](https://godotengine.org/)。这个小巧又免费的引擎也算是个杰作，之前我都在学习它，并且在上面写了一些demo，也算是入了门。

不过，godot的开发模式是，使用它提供的精美小巧的编辑器，并且写一门叫做`GDScript`的专用语言。
平心而论，编辑器和语言等等都设计得非常非常好。看得出来他们是认真为开发者考虑，把一些常用的功能都做得很方便了。并且还在不断拓展。通过它的[GDNative](https://godotengine.org/article/dlscript-here)，也能用c++，rust之类的语言来写代码，来解决性能瓶颈等等。

但是，就是这么一个小小的但是，我使用它的过程中，总缺少那么一点，怎么说呢，愉悦。对，愉悦！

## 合二为一的动机

用简单的方法解决复杂的事情本是最好的路子，但是现在在godot里面操作节点已经违背了我坚持rust的方针。这矛盾撞击的声音不断在脑子里面回响，搞得我心神不宁。

于是我打开了[arewegameyet](http://arewegameyet.com/categories/engines/)，开始选择rust的引擎，走上这条不归路（其实可以归）

它有啥吊的
======

回顾[上一个链接](http://arewegameyet.com/categories/engines/)，其中的网页列举了rust生态中当下的一些游戏引擎。如果数一下星星数，能发现数量最多的是piston，其次就是amethyst和ggez不相上下。

当时我看这个网页的时候，鬼使神差地忽略了piston（或许下期调研一下piston再补充说明一下），直接开始比较ggez和amethyst。

这两个的区别比较明显，ggez主打简单快速，amethyst明显更加庞大。

不过说实话我没有实际尝试过ggez，只看过其官网给出的简单例子。上面的区别也是我浅薄的总结。如果你有详细的信息，欢迎补充说明。

amethyst从结构上看就给人一种严谨的感觉，让我觉得他们这群人是真的要干大事的。

它集成了ecs模式[specs](https://slide-rs.github.io/specs/)，基于[gfx-hal](https://github.com/gfx-rs/gfx)搭建了自己的渲染系统[rendy](https://github.com/amethyst/rendy)，做了一套prefab功能。还撰写了详细的文档。

虽然我还没有认真对比研究它的基础设施于其他可替代的技术的优势，不过简单看还是很厉害的：specs提供了免费并行的高效ecs；gfx-hal支持vulkan，metal，DirectX11-12，OpenGL2.1/ES2+，支持2d和3d游戏。

同时，观察它的引用和依赖关系，并非是把一堆东西简单地组合在一起，而是自己开发了非常多的东西，把这些基础设施融入到自身来。

使用经历和感受
======

吹了那么多，其实客观上它还只是开荒阶段。从它的[roadmap](https://amethyst.rs/roadmap)可以看出，很多功能还在计划中。比如安装和ios要到年底才支持。

不过我想，等我用得滚瓜烂熟的时候，它应该也发展得差不多了……

下面列举一些感受

## 难学

rust本身的学习曲线已经够陡了，差不多一年了我才能够舒服地使用。amethyst的曲线也比较陡。第一个主要的阻碍应该就是specs的ecs了。

要融入它就得接受它的架构，整个逻辑不能简单地一个主循环搞定。然后就被迫开始熟悉那堆概念——`World`,`Component`,`System`,`Entity`,`Storage`,`Resource`...

另一个槽点是hello world足够复杂。用它的cli工具，new出来的新工程，长这个样子：

``` rust
use amethyst::{
    core::transform::TransformBundle,
    ecs::prelude::{ReadExpect, Resources, SystemData},
    prelude::*,
    renderer::{
        pass::DrawShadedDesc,
        rendy::{
            factory::Factory,
            graph::{
                render::{RenderGroupDesc, SubpassBuilder},
                GraphBuilder,
            },
            hal::{format::Format, image},
        },
        types::DefaultBackend,
        GraphCreator, RenderingSystem,
    },
    utils::application_root_dir,
    window::{ScreenDimensions, Window, WindowBundle},
};

struct MyState;

impl SimpleState for MyState {
    fn on_start(&mut self, _data: StateData<'_, GameData<'_, '_>>) {}
}

fn main() -> amethyst::Result<()> {
    amethyst::start_logger(Default::default());

    let app_root = application_root_dir()?;

    let resources_dir = app_root.join("resources");
    let display_config_path = resources_dir.join("display_config.ron");

    let game_data = GameDataBuilder::default()
        .with_bundle(WindowBundle::from_config_path(display_config_path))?
        .with_bundle(TransformBundle::new())?
        .with_thread_local(RenderingSystem::<DefaultBackend, _>::new(
            RenderingGraph::default(),
        ));

    let mut game = Application::new(resources_dir, MyState, game_data)?;
    game.run();

    Ok(())
}

#[derive(Default)]
struct RenderingGraph {
    dimensions: Option<ScreenDimensions>,
    surface_format: Option<Format>,
    dirty: bool,
}

impl GraphCreator<DefaultBackend> for RenderingGraph {
    fn rebuild(&mut self, res: &Resources) -> bool {
        // Rebuild when dimensions change, but wait until at least two frames have the same.
        let new_dimensions = res.try_fetch::<ScreenDimensions>();
        use std::ops::Deref;
        if self.dimensions.as_ref() != new_dimensions.as_ref().map(|d| d.deref()) {
            self.dirty = true;
            self.dimensions = new_dimensions.map(|d| d.clone());
            return false;
        }

        self.dirty
    }

    fn builder(
        &mut self,
        factory: &mut Factory<DefaultBackend>,
        res: &Resources,
    ) -> GraphBuilder<DefaultBackend, Resources> {
        use amethyst::renderer::rendy::{
            graph::present::PresentNode,
            hal::command::{ClearDepthStencil, ClearValue},
        };

        self.dirty = false;
        let window = <ReadExpect<'_, Window>>::fetch(res);
        let surface = factory.create_surface(&window);
        // cache surface format to speed things up
        let surface_format = *self
            .surface_format
            .get_or_insert_with(|| factory.get_surface_format(&surface));
        let dimensions = self.dimensions.as_ref().unwrap();
        let window_kind =
            image::Kind::D2(dimensions.width() as u32, dimensions.height() as u32, 1, 1);

        let mut graph_builder = GraphBuilder::new();
        let color = graph_builder.create_image(
            window_kind,
            1,
            surface_format,
            Some(ClearValue::Color([0.34, 0.36, 0.52, 1.0].into())),
        );

        let depth = graph_builder.create_image(
            window_kind,
            1,
            Format::D32Sfloat,
            Some(ClearValue::DepthStencil(ClearDepthStencil(1.0, 0))),
        );

        let opaque = graph_builder.add_node(
            SubpassBuilder::new()
                .with_group(DrawShadedDesc::new().builder())
                .with_color(color)
                .with_depth_stencil(depth)
                .into_pass(),
        );

        let _present = graph_builder
            .add_node(PresentNode::builder(factory, surface, color).with_dependency(opaque));

        graph_builder
    }
}
```

虽然说这个是hello world有失公正，但是它的功能又比较简单，还不能算脚手架，有点不伦不类。

观察一下可以发现，里面多数内容都在做渲染方面的初始化，甚至还没搞ecs部分。不过这就已经够劝退新人的了。

## 编译速度

rust编译是真的慢（官方在不断改进，今年已经比去年快了50%了），特别这种依赖很多的大工程，初次编译慢到怀疑人生。

之后的小修改编译也有10s级别，我的电脑可是8700k。

但是反回来说几句公道话，编辑器中rls插件倒可以在1s量级给出错误提示或告诉我能过编译。不老开游戏的话，开发体验还行。

## 错误提示

作为初学者，最恼人的错误就是开游戏马上panic，然后它给出的提示只是简单的“有资源没初始化”，然后告诉我，要看详细信息，请开nightly（rust区别于stable的较新版本）功能。

开完nightly，编辑器中的rls又编译不过了，倒腾过一阵也没倒腾成功。最后发现一个粗暴的小办法。

``` toml
# ...
[dependencies]
# ...

#####
# shred = {version = "*", features = ["nightly"]}
##### 看上面这行

# ...
```

平时在编辑器里注释掉`shred`的nightly功能，编译也用普通编译，只有出了panic不知道怎么查的时候才取消注释一下。

勉强也能苟住。

总结
====

这篇就吐槽这么多吧。总结一下。

### TL;DL

博主专(d)注(d)rust，又要做游戏，根据个人对各项目的未来判断，最终选了amethyst，跳入了无底大坑。


下一篇或许看一下piston，或许讲一下amethyst的简单项目代码。

咕咕咕