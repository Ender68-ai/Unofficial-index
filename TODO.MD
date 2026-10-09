TODO for your mod
1. Detect ModsLayer

    Listen for EnterLayerEvent filtered on "ModsLayer" (the ID set in ModsLayer::init) or use Alpha's $nodeModify(MyModsLayer, ModsLayer) + modify() ModsLayer.cpp:311-315

2. Add your tab button

    getChildByID("tabs-menu"), addChild a CCMenuItemSpriteExtra (a GeodeTabSprite-style sprite if you want it to match), setID("index2-button"_spr), then mainTabs->updateLayout() (it uses SimpleRowLayout with AxisScaling::Scale) ModsLayer.cpp:528-558
    Your button uses your own callback — ModsLayer::onTab/gotoTab and m_tabs are loader internals, so Geode's tabs won't deselect when yours is clicked; manage selection visuals yourself ModsLayer.cpp:692-698

3. Swap list content on click

    In your callback: layer->getChildByID("mod-list-frame"), remove the current ModList child (like gotoTab does with removeFromParent), and addChild your custom list node sized to frame->getContentSize() - ccp(30, 0) ModsLayer.cpp:700-715
    Consider hiding "page-menu" — ModsLayer::updateState() re-shows it via setVisible(true) on every page load ModsLayer.cpp:760-778

4. Fetch your index

    Use web::WebRequest + m_listener.spawn(...) — same pattern as server::getMods (params like page/per_page are 1-based server-side, 0-based locally) Server.cpp:699-737
    Parse the JSON yourself; if your index mirrors Geode's { "payload": [...] } envelope, copy the parseServerPayload pattern Server.cpp:233-246

5. Render items

    Build your own ScrollLayer/ListView with custom item nodes — ModList/ModItem/ModSource are loader/src internals not exported to mods, so you can't reuse them ModList.hpp:24-35
    Handle loading/empty states yourself (equivalent of LoadPageError/getNoModsFoundError) ModListSource.cpp:22-31

Watch out for

    ModsLayer::keyDown calls m_lists.at(m_currentSource) on Left/Right — don't touch m_currentSource, your node stays untouched by it ModsLayer.cpp:726-744
    tabs-menu has setTouchPriority(-150), so buttons inside it are safe from list touches ModsLayer.cpp:534-535

Notes

    What you lose vs. native ModList: pagination/search UI, install buttons, and ModDownloadManager progress integration — all of that would need to be rebuilt in your items.
    If you later want installs to go through Geode's download pipeline, server::ModDownloadManager and geode::openInfoPopup(modID) exist, but they resolve against the official index, not yours GeodeUI.cpp:188-207 .
