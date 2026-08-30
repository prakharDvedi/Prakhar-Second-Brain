- ## Prakhar
    
     [PEAR],  _—_ Yesterday at 8:54 PM
    
    evening boss [https://github.com/pear-protocol/agent-pear-v2/pull/640](https://github.com/pear-protocol/agent-pear-v2/pull/640 "https://github.com/pear-protocol/agent-pear-v2/pull/640") Had to merge this to main. related to the SignalProcessor errors basically Elfa detects the crossing, but by the time the webhook reaches us and we recheck the condition, the value might have already crossed back. So its more of a timing issue than a race PR doesnt change it, just adds morelogging to identify
    
- ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 1:04 PM
    
    got it.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 12:58 PM
    
    we have a meeting on monday as well for direction
    
- ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 12:58 PM
    
    ahh okay. any decisions made yet regardin new architecture?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 12:55 PM
    
    hmm make sense. i didnt want this to occur again. for v1 this would do.. ok. ill raise pr
    
- ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 12:51 PM
    
    gm ser ser, mapped the stale data paths from the categories incident. right now we have timestamps on the datasets but nothing actually checks their age, and most degraded refresh paths are warn only. planning a small freshness monitor that periodically checks the existing timestamps and alerts once when a dataset goes stale, daily while it stays stale, and on recovery. detection/alerts only first, no change to refresh or trading behavior. Thinking of working this. ill share entire plan on pear chat if you think this is right thing to do..
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:45 PM
    
    ah okay ![😅](https://discord.com/assets/5134d215343b97ef.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:43 PM
    
    Forwarded
    
    the script's ready boss. i have ran a dry run. correctly polls all 67 eligible users and connected users to tg.
    
    [](https://cdn.discordapp.com/attachments/1519254370397589567/1541858082072567898/image.png?ex=6a950d7d&is=6a93bbfd&hm=b6e06017fedb03e36a5f20f2983c1566ae66051e93f78fe231cdc7e2e62ffd67&)
    
    ![Image](https://media.discordapp.net/attachments/1519254370397589567/1541858082072567898/image.png?ex=6a950d7d&is=6a93bbfd&hm=b6e06017fedb03e36a5f20f2983c1566ae66051e93f78fe231cdc7e2e62ffd67&=&format=webp&quality=lossless&width=640&height=56)
    
    ```
    import "../src/load-indexing-env";
    import { createHash } from "node:crypto";
    import { AgentPrismaService, REDIS } from "@agent-pear/shared";
    import { BullModule } from "@nestjs/bullmq";
    import { type DynamicModule, Module } from "@nestjs/common";
    import { ConfigModule } from "@nestjs/config";
    ```
    
    send-all-notification.ts
    
    12 KB
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:43 PM
    
    1 sec
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:43 PM
    
    jeyd did
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:43 PM
    
    i didnt do it
    
-  [PEAR], 
    
    ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:43 PM
    
    i have
    
-  [PEAR], 
    
    ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:43 PM
    
    yes ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:22 PM
    
    i had to pitch my hackathon project in day so got slow a bit ![😓](https://discord.com/assets/7a079c219a80e127.svg) will cover it
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:20 PM
    
    i have some fixes in local ill push them if they are still valid. related to the things i found in tg bot response. i think now that kb have been updated more they would be reduced as weel
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:19 PM
    
    ah makes sense
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:15 PM
    
    ah
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:14 PM
    
    ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg) lighter reviews seem fine. one asset universe bug i saw today about a missing endpoint of category. pinged jeyd for it
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:13 PM
    
    ooh. got it. biz it is
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:12 PM
    
    or that's postponed again ![😓](https://discord.com/assets/7a079c219a80e127.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:12 PM
    
    like what they want ap to be
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:12 PM
    
    the one about ur call with cl
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:12 PM
    
    i was also interested in that part of AP direction
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:12 PM
    
    ahh biz
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:11 PM
    
    lol
    - anything new integration possibly?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:10 PM
    
    [](https://klipy.com/gifs/kid-awkward-7)
    
    ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAQxklEQVR4AQCBAH7/AMBzbf+/c23/vnNt/7x0bf+6dG7/t3Vu/7V1bv+xdm//rnZv/6p2b/+ndm//pHZv/6B2b/+ddm//m3Vu/5l0bv+Xc23/lnFs/5Vva/+UbWn/lGto/5RpZ/+VZ2X/lmRk/5diYv+YYGH/mV5g/5pcXv+bW17/m1pd/5xZXP+cWVz/AIEAfv8AwHNt/79zbf++dG3/vHRu/7p0bv+3dW7/tHVv/7F2b/+udm//qndv/6d3b/+jd2//oHZv/512b/+bdW7/mHRu/5dzbf+VcWz/lW9r/5Rtav+Ua2j/lGln/5VnZf+VZWT/lmJi/5dgYf+YXmD/mV1f/5pbXv+bWl3/m1ld/5xZXP8AgQB+/wC/dG7/v3Ru/710bv+8dG7/uXVu/7d1b/+0dm//sXZv/613cP+qd3D/pndw/6N3cP+gd3D/nXZv/5p2b/+YdW7/lnNt/5VybP+UcGv/k25q/5Nsaf+Uamf/lGdm/5VlZP+WY2P/l2Fh/5hfYP+ZXV//mlxe/5paXf+bWl3/m1ld/wCBAH7/AL91bv++dW//vXVv/7t1b/+5dm//tnZw/7N3cP+wd3D/rXhw/6l4cf+meHH/onhx/594cP+cd3D/mXZw/5d1b/+VdG7/lHJt/5NxbP+Tb2v/k2xp/5NqaP+TaGb/lGZl/5VjY/+WYWL/l19h/5heYP+ZXF//mlte/5paXv+aWl3/AIEAfv8AvnZv/712cP+8dnD/unZw/7h3cP+1d3D/snhx/694cf+seHH/qHly/6V5cv+heXL/nnlx/5t4cf+Zd3H/lnZw/5V1b/+Tc27/knFt/5JvbP+SbWr/kmtp/5JpZ/+TZmb/lGRk/5ViY/+WYGL/l15h/5hdYP+YXF//mVte/5lbXv8AgQB+/wC9d3H/vHdx/7t3cf+5d3H/t3hx/7R4cv+xeXL/rnly/6t6c/+nenP/pHpz/6B6c/+denP/mnly/5d4cv+Vd3H/k3Zw/5J0b/+Rcm7/kXBt/5Bua/+RbGr/kWpo/5JnZ/+TZWX/lGNk/5VhY/+WX2L/ll5h/5ddYP+YXF//mFxf/wCBAH7/ALx4cv+7eHL/unly/7h5cv+2eXP/s3pz/7B6c/+te3T/qnt0/6Z7dP+je3T/n3t0/5x7dP+ZenT/lnpz/5R4cv+Sd3H/kXVw/5B0b/+Pcm7/j29t/49ta/+Qa2r/kGho/5FmZ/+SZGX/k2Jk/5RgY/+VX2L/ll5h/5ZdYf+WXWD/AIEAfv8Au3p0/7p6dP+5enT/t3p0/7V7dP+ye3X/r3x1/6x8df+pfXX/pX12/6F9dv+efXX/m3x1/5h8df+Ve3T/k3p0/5F4c/+Pd3L/jnVx/45zb/+OcW7/jm5s/45sa/+Pamn/kGdo/5FlZv+SY2X/k2Jk/5NgY/+UX2L/lV5i/5VeYv8AgQB+/wC6e3X/uXx1/7h8df+2fHb/tH12/7F9dv+ufXb/q353/6d+d/+kfnf/oH53/51+d/+Zfnf/ln12/5R8dv+Re3X/j3p0/454c/+NdnL/jHRx/4xyb/+McG7/jW1s/41ra/+OaWn/j2do/5BlZ/+RY2X/kmFk/5JgZP+TX2P/k19j/wCBAH7/ALl9d/+4fXf/t353/7V+d/+zfnj/sH94/61/eP+qgHj/poB5/6OAef+fgHn/m4B5/5iAeP+Vf3j/kn53/5B9d/+OfHb/jXp1/4t4dP+LdnL/i3Rx/4txb/+Lb27/jGxs/41qa/+NaGn/jmZo/49kZ/+QY2b/kWJl/5FhZf+RYGT/AIEAfv8AuH95/7d/ef+2gHn/tIB5/7KAef+vgXr/rIF6/6mBev+lgnr/oYJ6/56Cev+agnr/l4F6/5SBev+RgHn/j394/419d/+Le3b/inp1/4l3dP+JdXL/iXNx/4pwb/+Kbm7/i2xs/4xpa/+NZ2n/jmZo/45kZ/+PY2b/j2Jm/5BiZv8AgQB+/wC3gXr/toF6/7WBe/+zgnv/sYJ7/66De/+rg3v/qIN8/6SEfP+ghHz/nYR8/5mEfP+Wg3z/k4J7/5CBe/+NgHr/i395/4p9eP+Je3f/iHl1/4h3dP+IdHL/iHJx/4lvb/+JbW3/imts/4tpa/+MZ2r/jWVp/41kaP+OZGf/jmNn/wCBAH7/ALaDfP+2g3z/tIN8/7OEfP+whH3/roR9/6qFff+nhX3/o4V+/6CGfv+chn7/mIV+/5WFff+ShH3/j4N8/4yCe/+KgHr/iX95/4h9eP+He3f/hnh1/4Z2dP+Hc3L/h3Fw/4hub/+JbG3/imps/4toa/+LZ2r/jGZp/4xlaf+NZGj/AIEAfv8AtoV+/7WFfv+0hX7/soZ+/7CGfv+thn//qod//6aHf/+jh3//n4d//5uHf/+Yh3//lId//5GGfv+OhX7/i4R9/4mCfP+IgHv/h355/4Z8eP+Fenb/hXd1/4Z1c/+GcnL/h3Bw/4hub/+IbG3/iWps/4poa/+LZ2r/i2Zq/4tmav8AgQB+/wC1h3//tYd//7SHf/+yiID/r4iA/62IgP+piYD/pomA/6KJgf+eiYH/m4mB/5eJgP+TiID/kIiA/42Hf/+LhX7/iYR9/4eCfP+GgHv/hX55/4R7eP+EeXb/hXZ1/4V0c/+GcXH/h29w/4dtb/+Ia23/iWps/4lobP+KaGv/imdr/wCBAH7/ALWJgf+1iYH/tImB/7KJgf+vioH/rIqB/6mKgv+mi4L/oouC/56Lgv+ai4L/l4uC/5OKgf+QiYH/jYiA/4qHf/+IhX7/hoN9/4WBfP+Ef3r/hH15/4R6d/+EeHb/hHV0/4Vzcv+GcHH/hm5w/4dsbv+Ia23/iGpt/4lpbP+JaGz/AIEAfv8AtouC/7WLgv+0i4L/souC/6+Lg/+tjIP/qYyD/6aMg/+ijYP/no2D/5qMg/+XjIP/k4yD/5CLgv+NioH/ioiB/4iHf/+GhX7/hYN9/4SAe/+Dfnr/g3t4/4N5d/+EdnX/hHRz/4Vxcv+Gb3H/h21v/4dsbv+Ia27/iGpt/4hpbf8AgQB+/wC2jIP/tYyD/7SNg/+yjYP/sI2E/62NhP+qjoT/po6E/6KOhP+ejoT/mo6E/5eOhP+TjYT/kIyD/42Lgv+KioL/iIiA/4aGf/+EhH7/hIJ8/4N/e/+DfXn/g3p3/4N3dv+EdXT/hXNz/4Vwcf+Gb3D/h21v/4dsbv+Ia27/iGpt/wCBAH7/ALaOhP+2joT/tY6E/7OOhP+wj4X/rY+F/6qPhf+mj4X/o5CF/5+Qhf+bj4X/l4+F/5OOhf+QjYT/jYyD/4qLgv+IiYH/hoeA/4WFf/+Eg33/g4B7/4N+ev+De3j/g3h2/4R2df+EdHP/hXFy/4Zvcf+GbnD/h21v/4dsbv+Ia27/AIEAfv8At4+F/7ePhf+1kIX/s5CF/7GQhv+ukIb/q5GG/6eRhv+jkYb/n5GG/5uRhv+YkIb/lJCF/5CPhf+NjYT/i4yD/4iKgv+GiIH/hYZ//4SEfv+DgXz/g396/4N8ef+DeXf/hHd1/4R0dP+FcnL/hnBx/4ZvcP+Hbm//h21v/4hsb/8AgQB+/wC4kYb/uJGG/7aRhv+0kYb/spGG/6+Shv+skof/qJKH/6SSh/+gkof/nJKH/5iRhv+VkYb/kZCF/46PhP+LjYT/iYuC/4eJgf+Fh4D/hIV+/4SCff+Df3v/g315/4R6d/+EeHb/hXV0/4Vzc/+GcXL/h29x/4ducP+HbW//iG1v/wCBAH7/ALmShv+5kob/t5KG/7WSh/+zk4f/sJOH/62Th/+pk4f/pZOH/6GTh/+dk4f/mZKH/5WShv+SkYb/j4+F/4yOhP+JjIP/h4qC/4aIgP+FhX7/hIN9/4SAe/+Efnn/hHt4/4R4dv+FdnT/hnRz/4Zycv+HcHH/h29w/4hub/+Ibm//AIEAfv8Au5OH/7qTh/+5k4f/t5OH/7SUh/+xlIf/rpSH/6qUiP+mlIj/opSI/56Uh/+ak4f/lpOH/5OShv+QkIX/jY+E/4qNg/+Ii4L/h4mA/4WGf/+FhH3/hIF7/4R+ev+Fe3j/hXl2/4Z2df+GdHP/h3Jy/4dxcf+Ib3D/iG9w/4hub/8AgQB+/wC8lIf/u5SH/7qUh/+4lIf/tZSH/7KViP+vlYj/q5WI/6eViP+jlYj/n5WI/5uUh/+Xk4f/lJKG/5GRhf+Oj4T/i46D/4mMgv+HiYD/hod//4WEff+FgXv/hX96/4V8eP+GeXb/hnd1/4d1c/+Hc3L/iHFx/4lwcP+Jb3D/iW5v/wCBAH7/AL2Vh/+8lYf/u5WH/7mViP+3lYj/tJWI/7CWiP+sloj/qZaI/6SWiP+glYj/nJWH/5iUh/+Vk4b/kpKF/4+QhP+MjoP/ioyC/4iKgP+Hh3//hoV9/4aCe/+Gf3r/hnx4/4Z6dv+Hd3X/iHVz/4hzcv+JcXH/iXBw/4pvb/+Kb2//AIEAfv8AvpWH/76Vh/+8lYj/upaI/7iWiP+1loj/sZaI/66WiP+qloj/ppaI/6GWiP+dlYj/mZSH/5aThv+Skob/kJCF/42Pg/+LjIL/iYqA/4iIf/+HhX3/h4J7/4d/ev+HfXj/h3p2/4h4df+IdXP/iXNy/4lycf+KcHD/inBv/4pvb/8AgQB+/wC/loj/v5aI/72WiP+7loj/uZaI/7aXiP+yl4j/r5eI/6uXiP+nl4j/opaI/56WiP+alYf/l5SG/5OShv+QkYT/jo+D/4yNgv+KioD/iYh//4iFff+Hgnv/h4B5/4h9eP+Ienb/iHh0/4l2c/+KdHL/inJx/4txcP+LcG//i29v/wCBAH7/AMCWiP/Aloj/vpaI/7yXiP+6l4j/t5eI/7OXiP+wl4j/rJeI/6iXiP+jl4j/n5aI/5uVh/+YlIb/lJOF/5GRhP+Pj4P/jY2C/4uLgP+KiH//iYV9/4iDe/+IgHn/iH14/4l6dv+JeHT/inZz/4p0cf+LcnD/i3Fw/4twb/+Mb2//AIEAfv8AwZeI/8GXiP+/l4j/vZeI/7uXiP+4l4j/tJiI/7GYiP+tmIj/qJeI/6SXiP+gloj/nJaH/5iUhv+Vk4X/kpGE/4+Pg/+NjYL/jIuA/4qIf/+Jhn3/iYN7/4mAef+JfXf/iXt2/4p4dP+KdnP/i3Rx/4tycP+McW//jHBv/4xwb/8AgQB+/wDCl4j/wZeI/8CXiP++l4j/u5eI/7iYiP+1mIj/sZiI/62YiP+pmIj/pZeI/6GXh/+dlof/mZWG/5aThf+TkoT/kJCD/46Ngv+Mi4D/i4h+/4qGff+Jg3v/iYB5/4l9d/+Ke3b/inh0/4t2c/+LdHH/jHJw/4xxb/+McG//jXBu/wCBAH7/AMOXiP/Cl4j/wJeI/7+XiP+8mIj/uZiI/7WYiP+ymIj/rpiI/6mYiP+ll4j/oZeH/52Wh/+ZlYb/lpOF/5OShP+QkIP/jo6C/4yLgP+LiX7/ioZ9/4qDe/+JgHn/in13/4p7dv+KeHT/i3Zy/4t0cf+McnD/jHFv/41wb/+NcG7/AYEAfv8Aw5eI/8KXiP/Bl4j/v5eI/7yYiP+5mIj/tpiI/7KYiP+umIj/qpiI/6WXiP+hl4f/nZaH/5qVhv+Wk4X/k5KE/5GQg/+OjoL/jYuA/4uJfv+Khn3/ioN7/4qAef+KfXf/int1/4t4dP+LdnL/jHRx/4xycP+NcW//jXBv/41wbv9t4TlhH3KYOgAAAABJRU5ErkJggg==)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:10 PM
    
    [](https://cdn.discordapp.com/attachments/1519254370397589567/1541849821193707661/image.png?ex=6a9505cb&is=6a93b44b&hm=07c5f8b73bd8c460885b1eb6460a2c678756909e11f319bee4281a7075c03ae9&)
    
    ![Image](https://media.discordapp.net/attachments/1519254370397589567/1541849821193707661/image.png?ex=6a9505cb&is=6a93b44b&hm=07c5f8b73bd8c460885b1eb6460a2c678756909e11f319bee4281a7075c03ae9&=&format=webp&quality=lossless&width=452&height=512)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABsAAAAgCAYAAADjaQM7AAAORklEQVR4AQBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wBtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/wFtAJL/ACUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/yUpK/8lKSv/JSkr/8T+9T31XzmRAAAAAElFTkSuQmCC)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:10 PM
    
    if it's alr
    
- ## Prakhar
    
     [PEAR],  _—_ 8/25/2026 10:09 PM
    
    ser call with russian leads ended? whats happenin
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:22 PM
    
    any guess or info?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:22 PM
    
    oh ok ![😓](https://discord.com/assets/7a079c219a80e127.svg) when its being launched exactly?
    
    ⁠PEAR⁠
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:19 PM
    
    i feel we are solid
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:19 PM
    
    on our agent side
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:19 PM
    
    ill keep an eye as well ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:17 PM
    
    yes sure
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:13 PM
    
    yeah.. jeyd mentioned he had to run some scripts to populate data and prices
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 6:10 PM
    
    my bad ser, i had bad internet in start and huf opened ap in beginning today, baskets and lighter, didnt get what was said. are they satisfied?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 2:00 PM
    
    Ohhh nice
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 12:05 PM
    
    yes
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 12:03 PM
    
    oh ok. ill see the remaining cleanups . plus that margin check pr once if you could see related to not placing order from agent pear when margin is not available instead of failing [https://github.com/pear-protocol/agent-pear-v2/pull/569](https://github.com/pear-protocol/agent-pear-v2/pull/569 "https://github.com/pear-protocol/agent-pear-v2/pull/569")
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 12:01 PM
    
    true.. we'll have to update the asset library right?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 12:00 PM
    
    yes these are some fixes. so is the trade execution one. ill actually be reviewing the codebase once to see stale hyperliquid mention, docs, etc. i think trading part is alr, you've also tested. morning i checked some pair and baskets seemed fine
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 11:50 AM
    
    gm ser ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 4:08 PM
    
    Thanks ser
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 4:08 PM
    
    [](https://klipy.com/gifs/evil-superman)
    
    ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAgCAYAAABzenr0AAAQxklEQVR4AQCBAH7/AMGlrv/Bpa3/waWs/8Gkqv/Co6f/wqKk/8OhoP/Dn5z/xJ2Y/8SblP/DmZD/w5aM/8KTiP/AkIX/vo2D/7yJgP+5hn//toJ+/7J+fv+uen7/qnZ//6ZygP+iboH/nWqD/5lnhf+VY4f/kWGJ/45ei/+MXIz/iluO/4hajv+HWY//AIEAfv8AwKWt/8Ckrf/BpKv/waOp/8Kjpv/CoqP/w6Cf/8OfnP/DnZf/w5uT/8OYj//Clov/wZOI/8CQhf++jIL/vImA/7mFfv+2gX3/sn19/655ff+qdX7/pnF//6Ftgf+daoP/mWaE/5Vjhv+RYIj/jl6K/4xcjP+JWo3/iFmO/4dZjv8AgQB+/wC/pKz/wKOs/8Cjqv/Aoqj/waKl/8Ghov/Cn57/wp6b/8Kclv/DmpL/wpiO/8KViv/Bkof/v4+E/72Mgf+7iH//uIR9/7WBfP+yfXz/rnl8/6p1ff+lcX7/oW2A/51pgf+YZoP/lWOF/5Fgh/+OXYn/i1uL/4lajP+IWY3/h1iN/wCBAH7/AL6iq/++oqr/v6Kp/7+hp//AoKT/wJ+h/8Genf/BnJn/wZuV/8GZkf/Blo3/wZSJ/8CRhf++joL/vIp//7qHff+3g3z/tIB7/7F8ev+teHv/qXR7/6Vwff+gbH7/nGiA/5hlgv+UYoT/kV+G/41dh/+LW4n/iVmK/4dYi/+HWIz/AIEAfv8AvKCp/72gqP+9oKf/vZ+l/76eov++nZ//v5yb/8Cbl//AmZP/wJeP/8CVi/+/kof/vo+D/72MgP+7iX3/uYZ7/7aCev+zfnn/sHp4/6x2ef+ocnn/pG96/6BrfP+bZ37/l2SA/5Nhgv+QXoT/jVyF/4pah/+IWIj/h1eJ/4ZXif8AgQB+/wC6nqf/up6m/7udpP+7naL/vJyg/7ybnP+9mpn/vpiV/76Xkf++lYz/vpKI/72QhP+9jYH/u4p+/7qHe/+4hHn/tYB3/7J8dv+ueXb/q3V2/6dxd/+jbXj/nml5/5pme/+WYn3/kl9//49dgf+MWoP/iViE/4dXhv+GVob/hVWH/wCBAH7/ALibpP+4m6P/uJui/7maoP+5mZ3/upia/7qXlv+7lpL/u5SO/7ySiv+8kIb/u46C/7qLfv+5iHv/uIV4/7aCdv+zfnT/sHp0/613c/+pc3P/pW90/6Frdf+daHf/mWR4/5Vhev+RXnz/jlt+/4tZgP+IV4H/hlWD/4VUhP+EVIT/AIEAfv8AtJih/7WYoP+1mJ//tZed/7aWmv+3lZf/t5ST/7iTj/+5kYv/uY+H/7mNg/+5i3//uIh7/7eGeP+1gnX/s39z/7F8cf+ueHD/q3Vw/6dxcP+jbXH/n2ly/5tmc/+XYnX/k193/5Bcef+MWXv/iVd9/4dVfv+FVH//hFOA/4NSgf8AgQB+/wCxlZ7/sZSd/7GUnP+ylJr/spOX/7OSlP+0kZD/tZCM/7WOiP+2jIT/top//7WIfP+1hXj/tIN1/7KAcv+wfXD/rnlu/6t2bf+ocm3/pW9t/6Frbv+dZ2//mWRw/5Vgcv+RXXT/jlp2/4pYeP+HVXn/hVR7/4NSfP+CUX3/gVF9/wCBAH7/AKyRm/+tkZr/rZCZ/62Qlv+uj5T/r46Q/7CNjf+xjIn/sYuF/7KJgP+yh3z/soV4/7GCdf+wgHH/r31v/616bP+rd2v/qHNq/6Vwaf+ibGn/nmhq/5tla/+XYW3/k15u/49bcP+LWHL/iFZ0/4VTdv+DUnf/gVB5/4BPef9/T3r/AIEAfv8Ap42Y/6iNl/+ojZX/qYyT/6mLkP+qi43/q4qJ/6yIhf+th4H/rYV9/66Eef+ugXX/rX9x/6x9bv+remv/qXdp/6d0Z/+lcGb/om1m/59pZv+bZmf/l2No/5Rfaf+QXGv/jFlt/4lWb/+FVHD/g1Fy/4BQdP9+TnX/fU12/31Ndv8AgQB+/wCiiZT/oomU/6OIkv+jiJD/pIeN/6WHiv+mhob/p4WC/6iDfv+ognr/qYB2/6l+cv+ofG7/qHlr/6d2aP+ldGb/o3Fk/6FtY/+eamL/m2dj/5hjY/+UYGT/kF1m/41aZ/+JV2n/hVRr/4JRbf+AT2//fU5w/3tMcf96S3L/ektz/wCBAH7/AJyFkf+chJH/nYSP/52Ejf+eg4r/n4KH/6CCg/+hgX//on97/6N+d/+jfHL/o3pu/6N4a/+jdmf/onNl/6BwYv+fbWH/nGpg/5pnX/+XZF//k2Fg/5BdYf+MWmL/iVdk/4VUZv+CUmj/f09p/3xNa/96S23/eEpu/3dJb/92SW//AIEAfv8AlYCP/5WAjv+WgIz/l3+K/5h/h/+ZfoT/mn2A/5t8fP+ce3j/nXp0/514b/+edmv/nnRo/51yZP+ccGL/m21f/5lqXv+XZ1z/lWRc/5JhXP+PXl3/i1te/4hYX/+EVWH/gVJi/35PZP97TWb/eEto/3ZJaf90SGv/c0dr/3JHbP8AgQB+/wCOfIz/jnyL/497iv+Qe4f/kHuF/5J6gf+TeX7/lHh6/5V3df+WdnH/l3Rt/5dzaf+XcWX/l29i/5ZsX/+Valz/lGdb/5JkWv+PYVn/jF5Z/4lbWv+GWFv/g1Vc/4BSXv98UF//eU1h/3ZLY/90SWX/cUdm/3BGZ/9vRWj/bkVp/wCBAH7/AIZ3iv+Hd4n/h3eH/4h3hf+JdoL/inZ//4t1e/+NdHf/jnNz/49yb/+QcWr/kG9m/5FtYv+Qa1//kGlc/49nWv+NZFj/jGFX/4leVv+HW1b/hFlX/4FWWP9+U1n/elBb/3dNXf90S17/cUlg/29HYv9sRWP/a0Rl/2pDZf9pQ2b/AIEAfv8AfnOI/35zh/9/c4X/gHKD/4FygP+Ccn3/hHF5/4Vwdf+Gb3H/h25s/4htaP+Ja2T/iWpg/4loXf+JZlr/iGNY/4dhVv+FXlX/g1xU/4FZVP9+VlX/e1NV/3hQV/91Tlj/cUta/25JXP9sRl7/aUVf/2dDYf9mQmL/ZEFj/2RBY/8AgQB+/wB2b4b/dm+F/3duhP94boH/eW5//3pte/97bXf/fWxz/35rb/9/amv/gWlm/4FoYv+CZl7/gmRb/4JiWP+BYFb/gF5U/35bU/98WVL/elZS/3dTUv91UVP/ck5V/29LVv9rSVj/aUZa/2ZEXP9kQ13/YkFf/2BAYP9fP2H/Xj9h/wCBAH7/AG1qhP9taoT/bmqC/29qgP9wan3/cWl6/3Npdv90aHL/dmht/3dnaf95ZmX/emRh/3pjXf96YVn/el9W/3pdVP95W1L/d1lR/3VWUP9zU1D/cVFR/25OUv9rTFP/aElU/2VHVv9iRFj/YEJa/15BW/9cP13/Wj5e/1k9X/9YPV//AIEAfv8AZGaD/2Rmg/9lZoH/ZmZ//2dmfP9pZnn/amV1/2xlcf9uZGz/b2No/3BiZP9xYV//cmBc/3NeWP9zXFX/clpT/3FYUf9wVlD/blRP/2xRT/9qT0//Z0xQ/2RJUv9iR1P/X0VV/1xDV/9a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)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 3:36 PM
    
    Gm ser. Actually im a bit afk as i am transit for that hackathon i mentioned. Will be reaching room in some time ill try to continue then. Hope nothing is urgent. Huf seems to have trouble ![😕](https://discord.com/assets/4b78aed623919cf4.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 9:15 AM
    
    Gm ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 9:15 AM
    
    Yes les go
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 4:50 PM
    
    Got it. Will do
    - ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:43 PM
    
    got it
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:42 PM
    
    before?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:41 PM
    
    thanks for putting in word ser ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:40 PM
    
    yes i was informed
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:39 PM
    
    well i just told him im not looking so ended there with a heart
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:39 PM
    
    nice. can do that from now
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:37 PM
    
    ohh lol
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:35 PM
    
    i had this thought you guys must have been approached as well
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:34 PM
    
    ohh
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:34 PM
    
    for setting up call etc
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:34 PM
    
    their recruiter messaged
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:34 PM
    
    Gauntlet
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:34 PM
    
    no through discord
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:34 PM
    
    well they were not AI native
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:33 PM
    
    i got approached by 1 for recruiting
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:33 PM
    
    true. lets hope we settle
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:31 PM
    
    post lighter. ill handle the mantainance and fixing part. will give you time to decide more on architecture side
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:30 PM
    
    hope biz might not dump a random new feature all of a sudden. it's clear for now
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:29 PM
    
    ![📝](https://discord.com/assets/5f6b1b0353091184.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:26 PM
    
    its pretty optimised honestly i had to go to see videos on some advance stuff on llmops
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:25 PM
    
    yeah. well once this lighter part is done. can think on it as well
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:23 PM
    
    also. one thing i was working on side. how to reduce our open router costs. optimisations, any previous work or ideas on this?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:23 PM
    
    5 lpa one guy got in our clg TCS lol
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:22 PM
    
    thats much much more
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:22 PM
    
    ser ?
    - hh ok.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:21 PM
    
    5k in no world makes sense. can have a whole team who scans twitter whole day and give handmade triggers for that
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:19 PM
    
    ![😈](https://discord.com/assets/3c854880762932b9.svg) imagine we learn build our own and have it disposed lol once the free is gone
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:18 PM
    
    290 itself?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:18 PM
    
    500 trigger one?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:18 PM
    
    and for that
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:18 PM
    
    5k
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:18 PM
    
    holy
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:17 PM
    
    and + mantainance.. yeah
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:17 PM
    
    and for that a seperate repo? and the whole build part. we'd also have to tune things for it.. if we learn from elfa triggers while it active on our AP, understand it's triggers, what gets passed, where does it draw things from etc. a month or two for it maybe?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:12 PM
    
    true. yeah. well the biz guys are also about saving money so if they do hear we building this ![👀](https://discord.com/assets/4b6781044811b929.svg) might get them happy
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:11 PM
    
    i agree on this. we should not be dependent on a third party for this i feel . this trigger feature is so cool it might be one of top things about it
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:10 PM
    
    yes i remember reading this
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:06 PM
    
    if we were to build our own tho, we'll need feed collector which partially we have, store, then maybe a normaliser and intent parser.. ![🤔](https://discord.com/assets/d5d18f9c17a19de2.svg) how feasible might it be and how long might that take
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 2:04 PM
    
    true yeah. which wants me to have it id prefer being pinged as well for what i want. 500 triggers might be sufficient for now and jeyd's idea was that to have some global one and then have some freedom for users. i feel like that would work. but even still it's a dependency, we are at their limit's mercy. the ceiling is always theirs
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 1:55 PM
    
    i have mixed feelings but i'd kinda lean 60 no 40 yes. the feature's obv amazing and id surely want it, but i think scaling it, too much freedom and then mantaining the lifecyle of many triggers. spams, stale trigger plus the cost as well mainly. but at the same time its not like these arent solveable , we can have a per user limit, algorithm to decide who gets what when which alrdy has been implemented + the commands to turn them on off etc. till we are dependent on elfa i might say no, it could be a compounding cost for us, but once we kinda make smth similar we can have it added what do you say?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 12:59 PM
    
    gm! Pls run a review once on this as well. tg lighter and advanced mode PR [https://github.com/pear-protocol/agent-pear-v2/pull/564](https://github.com/pear-protocol/agent-pear-v2/pull/564 "https://github.com/pear-protocol/agent-pear-v2/pull/564")
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 10:33 PM
    
    - tg commands
    - advanced mode
    
    noted. ill be workin on tonight
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 3:33 PM
    
    prakhar@pearprotocol.io
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 2:00 PM
    
    yes commands that native hl default. ill take them in later pr
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 1:59 PM
    
    thnks ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 1:07 PM
    
    pheww
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 12:12 PM
    
    ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg) was on the review loop. will do
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 12:11 PM
    
    oh db snapshot
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 12:11 PM
    
    ![☠️](https://discord.com/assets/04de4da79291ee41.svg)
    
    [](https://cdn.discordapp.com/attachments/1519254370397589567/1539524730162249768/image.png?ex=6a952222&is=6a93d0a2&hm=f1c0805982b906c4e0f19240e85c6c1ee795fd2ffeca58a82f579c972d2d7892&)
    
- it just dropped details that would have been useful and yeah the fix is simple. rest is all set up
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 11:33 AM
    
    [https://github.com/pear-protocol/agent-pear-v2/issues/532](https://github.com/pear-protocol/agent-pear-v2/issues/532 "https://github.com/pear-protocol/agent-pear-v2/issues/532") well this is blocked because Lighter errors are structured inside the be connector but get squashed to plain strings before reaching Agent. We need a stable errorCode propagated through LegResult/API/core sdk first. don't want to regex backend error messages in Agent.  
    once that's done we can have it correctly on the User side
    
    [](https://cdn.discordapp.com/attachments/1519254370397589567/1539515041328537661/image.png?ex=6a95191c&is=6a93c79c&hm=32016f5da4005dd10efac4419d8f5216c4305583cd370444cb97fd14ed16f001&)
    
    ![Image](https://media.discordapp.net/attachments/1519254370397589567/1539515041328537661/image.png?ex=6a95191c&is=6a93c79c&hm=32016f5da4005dd10efac4419d8f5216c4305583cd370444cb97fd14ed16f001&=&format=webp&quality=lossless&width=320&height=320)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAXCAYAAABqBU3hAAAMEElEQVR4AQCBAH7/ACI/R/8iP0j/Ij9I/yI/SP8iPkj/Ij5I/yI+SP8iPkn/Ij5J/yE+Sf8hPkr/IT1K/yE9Sv8hPUv/ID1L/yA8TP8gPE3/IDxN/x88Tv8fO07/HztP/x47T/8eO1D/HjpQ/x46Uf8dOlH/HTpS/x05Uv8dOVL/HTlS/x05Uv8dOVP/AIEAfv8AIj9H/yI/R/8iP0f/Ij9I/yI/SP8iPkj/Ij5I/yI+SP8iPkn/Ij5J/yE+Sf8hPUr/IT1K/yE9S/8gPUv/IDxM/yA8TP8gPE3/HzxO/x87Tv8fO0//HjtP/x47UP8eOlD/HjpR/x46Uf8dOlH/HTpS/x05Uv8dOVL/HTlS/x05Uv8AgQB+/wAiP0f/Ij9H/yI/R/8iP0f/Ij9I/yI/SP8iPkj/Ij5I/yI+Sf8iPkn/IT5J/yE+Sv8hPUr/IT1L/yE9S/8gPUz/IDxM/yA8Tf8fPE3/HztO/x87Tv8fO0//HjtQ/x46UP8eOlH/HjpR/x06Uf8dOlL/HTlS/x05Uv8dOVL/HTlS/wCBAH7/ACM/R/8jP0f/Iz9H/yM/R/8iP0f/Ij9H/yI/SP8iPkj/Ij5I/yI+Sf8iPkn/IT5J/yE9Sv8hPUr/IT1L/yA9S/8gPEz/IDxN/yA8Tf8fPE7/HztO/x87T/8eO0//HjtQ/x46UP8eOlH/HjpR/x06Uf8dOlL/HTlS/x05Uv8dOVL/AIEAfv8AIz9H/yM/R/8jP0f/Iz9H/yM/R/8jP0f/Ij9H/yI/SP8iPkj/Ij5I/yI+Sf8iPkn/IT5K/yE9Sv8hPUv/IT1L/yA9TP8gPEz/IDxN/x88Tf8fO07/HztO/x87T/8eO1D/HjpQ/x46UP8eOlH/HjpR/x06Uf8dOlL/HTpS/x06Uv8AgQB+/wAjP0b/Iz9G/yM/Rv8jP0b/Iz9H/yM/R/8jP0f/Ij9H/yI/SP8iPkj/Ij5I/yI+Sf8hPkn/IT5K/yE9Sv8hPUv/ID1L/yA8TP8gPEz/IDxN/x88Tv8fO07/HztP/x47T/8eO1D/HjpQ/x46UP8eOlH/HjpR/x06Uf8dOlH/HTpR/wCBAH7/ACNARv8jQEb/Iz9G/yM/Rv8jP0b/Iz9G/yM/R/8jP0f/Ij9H/yI/SP8iPkj/Ij5I/yI+Sf8hPkn/IT1K/yE9Sv8hPUv/ID1M/yA8TP8gPE3/HzxN/x88Tv8fO07/HztP/x47T/8eO1D/HjpQ/x46UP8eOlH/HjpR/x46Uf8eOlH/AIEAfv8AI0BF/yNARf8jQEb/I0BG/yNARv8jP0b/Iz9G/yM/R/8jP0f/Ij9H/yI/SP8iPkj/Ij5I/yI+Sf8hPkn/IT1K/yE9S/8gPUv/ID1M/yA8TP8gPE3/HzxN/x87Tv8fO0//HztP/x47T/8eO1D/HjpQ/x46UP8eOlH/HjpR/x46Uf8AgQB+/wAkQEX/JEBF/yRARf8jQEX/I0BF/yNARv8jP0b/Iz9G/yM/Rv8jP0f/Ij9H/yI/SP8iPkj/Ij5J/yE+Sf8hPkr/IT1K/yE9S/8gPUv/IDxM/yA8Tf8fPE3/HzxO/x87Tv8fO0//HztP/x47UP8eOlD/HjpQ/x46UP8eOlD/HjpR/wCBAH7/ACRARf8kQEX/JEBF/yRARf8kQEX/I0BF/yNARv8jP0b/Iz9G/yM/R/8jP0f/Ij9H/yI+SP8iPkj/Ij5J/yE+Sf8hPUr/IT1L/yA9S/8gPUz/IDxM/yA8Tf8fPE7/HztO/x87T/8fO0//HjtP/x47UP8eOlD/HjpQ/x46UP8eOlD/AIEAfv8AJEBE/yRARf8kQEX/JEBF/yRARf8kQEX/I0BF/yNARv8jP0b/Iz9G/yM/R/8iP0f/Ij9I/yI+SP8iPkn/IT5J/yE9Sv8hPUr/IT1L/yA9TP8gPEz/IDxN/x88Tf8fO07/HztO/x87T/8eO0//HjtQ/x47UP8eOlD/HjpQ/x46UP8AgQB+/wAkQET/JEBE/yRARP8kQEX/JEBF/yRARf8jQEX/I0BG/yM/Rv8jP0b/Iz9H/yM/R/8iP0j/Ij5I/yI+Sf8hPkn/IT5K/yE9Sv8hPUv/ID1M/yA8TP8gPE3/HzxN/x88Tv8fO07/HztP/x87T/8eO0//HjtQ/x46UP8eOlD/HjpQ/wCBAH7/ACRARP8kQET/JEBE/yRARf8kQEX/JEBF/yRARf8jQEX/I0BG/yM/Rv8jP0f/Iz9H/yI/SP8iPkj/Ij5J/yE+Sf8hPkr/IT1K/yE9S/8gPUz/IDxM/yA8Tf8fPE3/HztO/x87Tv8fO0//HjtP/x47UP8eO1D/HjpQ/x46UP8eOlD/AIEAfv8AJEBE/yRARP8kQET/JEBF/yRARf8kQEX/I0BF/yNARf8jP0b/Iz9G/yM/R/8jP0f/Ij9I/yI+SP8iPkn/IT5J/yE9Sv8hPUr/IT1L/yA9TP8gPEz/IDxN/x88Tf8fO07/HztO/x87T/8eO0//HjtQ/x46UP8eOlD/HjpQ/x46UP8AgQB+/wAkQET/JEBE/yRARf8kQEX/JEBF/yRARf8jQEX/I0BG/yM/Rv8jP0b/Iz9H/yI/R/8iP0j/Ij5I/yI+Sf8hPkn/IT1K/yE9S/8gPUv/IDxM/yA8TP8gPE3/HzxO/x87Tv8fO0//HztP/x47UP8eOlD/HjpQ/x46UP8eOlD/HjpR/wCBAH7/ACRARf8kQEX/JEBF/yRARf8kQEX/I0BF/yNARf8jQEb/Iz9G/yM/R/8jP0f/Ij9H/yI+SP8iPkj/Ij5J/yE+Sv8hPUr/IT1L/yA9S/8gPEz/IDxN/x88Tf8fO07/HztO/x87T/8eO0//HjtQ/x46UP8eOlD/HjpR/x46Uf8eOlH/AIEAfv8AJEBF/yRARf8kQEX/JEBF/yNARf8jQEX/I0BG/yM/Rv8jP0b/Iz9H/yI/R/8iP0j/Ij5I/yI+Sf8hPkn/IT1K/yE9Sv8hPUv/ID1M/yA8TP8gPE3/HzxO/x87Tv8fO0//HjtP/x47UP8eOlD/HjpQ/x46Uf8eOlH/HjpR/x46Uf8AgQB+/wAkQEX/JEBF/yRARf8jQEX/I0BF/yNARv8jP0b/Iz9G/yM/R/8jP0f/Ij9H/yI+SP8iPkj/Ij5J/yE+Sv8hPUr/IT1L/yA9S/8gPEz/IDxN/x88Tf8fO07/HztO/x87T/8eO0//HjpQ/x46UP8eOlH/HjpR/x06Uf8dOlH/HTpR/wCBAH7/ACNARf8jQEX/I0BF/yNARv8jQEb/Iz9G/yM/Rv8jP0f/Iz9H/yI/R/8iP0j/Ij5I/yI+Sf8hPkn/IT5K/yE9Sv8hPUv/ID1M/yA8TP8gPE3/HzxO/x87Tv8fO0//HjtP/x47UP8eOlD/HjpR/x46Uf8dOlH/HTpS/x06Uv8dOlL/AIEAfv8AI0BG/yNARv8jQEb/I0BG/yM/Rv8jP0b/Iz9G/yM/R/8jP0f/Ij9I/yI+SP8iPkj/Ij5J/yE+Sv8hPUr/IT1L/yA9S/8gPEz/IDxN/x88Tf8fPE7/HztO/x87T/8eO1D/HjpQ/x46Uf8eOlH/HTpR/x06Uv8dOVL/HTlS/x05Uv8AgQB+/wAjQEb/I0BG/yNARv8jP0b/Iz9G/yM/Rv8jP0f/Iz9H/yI/R/8iP0j/Ij5I/yI+Sf8hPkn/IT5K/yE9Sv8hPUv/ID1M/yA8TP8gPE3/HzxN/x87Tv8fO0//HjtP/x47UP8eOlD/HjpR/x06Uf8dOlL/HTlS/x05Uv8dOVL/HTlS/wCBAH7/ACM/Rv8jP0b/Iz9G/yM/Rv8jP0b/Iz9H/yM/R/8jP0f/Ij9H/yI+SP8iPkj/Ij5J/yE+Sf8hPUr/IT1K/yE9S/8gPUz/IDxM/yA8Tf8fPE7/HztO/x87T/8eO0//HjpQ/x46UP8eOlH/HTpR/x06Uv8dOVL/HTlS/x05Uv8dOVL/AYEAfv8AIz9G/yM/Rv8jP0b/Iz9G/yM/Rv8jP0f/Iz9H/yI/R/8iP0j/Ij5I/yI+SP8iPkn/IT5J/yE9Sv8hPUv/IT1L/yA9TP8gPEz/IDxN/x88Tv8fO07/HztP/x47T/8eOlD/HjpR/x46Uf8dOlH/HTpS/x05Uv8dOVL/HTlS/x05U/8tG8G8cz5vtQAAAABJRU5ErkJggg==)
    
    [](https://cdn.discordapp.com/attachments/1519254370397589567/1539515041639039046/image.png?ex=6a95191c&is=6a93c79c&hm=da3aee829162dbba589bd5be53e9797b58f480698cecdb3df5d245da699b2a59&)
    
    ![Image](https://media.discordapp.net/attachments/1519254370397589567/1539515041639039046/image.png?ex=6a95191c&is=6a93c79c&hm=da3aee829162dbba589bd5be53e9797b58f480698cecdb3df5d245da699b2a59&=&format=webp&quality=lossless&width=320&height=241)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAJCAYAAABT2S4KAAAEvElEQVR4AQCBAH7/ACE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/AIEAfv8AIT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8AgQB+/wAhPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/wCBAH7/ACE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/AIEAfv8AIT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8AgQB+/wAhPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/wCBAH7/ACE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/AIEAfv8AIT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8BgQB+/wAhPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/yE9S/8hPUv/IT1L/2fR3RC3Hn9kAAAAAElFTkSuQmCC)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 11:18 AM
    
    on the BE side need some work post this as well. the error handling etc. non blocking though. will be testing it today once all is good for local boot up
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 11:17 AM
    
    yes. on it
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 11:14 AM
    
    yep better will share him the details
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 11:12 AM
    
    annd i checked the stats/quant side as well. lighter support there is a bit different from execution. the db/platform layer is already generic enough to add LIGHTER, and the rust calc side is mostly platform agnostic, so we can probably add lighter-listed assets + baskets using the existing shared HL/binance stats without a big rewrite. but fully native lighter analysis is a bigger thing. prices/pair stats right now are keyed by asset id, not venue, and we don't have a confirmed lighter historical candle pipeline either. so keeping separate HL vs lighter history/stats would need some proper source/venue separation first. and that is like big architecture change
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 10:38 AM
    
    **Quick Lighter update:** I am through most of the core Agent side correctness work now. Lighter is ungated in the integration branch, account/venue selection is deterministic, margin behavior is verified, executable thresholds now use the selected venue’s price instead of always anchoring to HL, and the trade exec prompt + KB are venue updated While doing the integration I also found and fixed two pre existing correctness issues that surfaced with multi-venue support: an explicit account could be used for a read and then a later write could fall back to another default account in the same turn, and stale asset cache prices could survive when symbols disappeared from later snapshots. Both are fixed with regression coverage. Main design decisions so far: execution venue comes from the resolved trade account, not model text SurfaceVenue and execution Connector stay separate no silent HL fallback for Lighter explicit user price levels are preserved; only Agent derived levels use venue native pricing changes are additive #532 failure UX is blocked until BE preserves stable Lighter error codes instead of matching error strings what’s left is mainly Lighter asset validation/indexing (#538), broader venue aware reads (#547), the live quantity check (#550), then the controlled full E2E (#531). So I’d say the static/code side integration is decently far along; the biggest remaining step is the live verification will be trying to work this out by today, do live testing. Please run reviews and suggestions on PR as well. (analysis and stats repo work side will be following up as well)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 9:32 AM
    
    Gm ser!. for those Elfa Pings - initial problem was that Elfa sends two different identifiers: a delivery eventId and an executionId. Our controller assumed they must be identical, so valid live notifications were rejected before reaching the queue. I initially changed the validation to allow those IDs to differ, normalized queryId from both payload locations, and kept eventId for deduplication. During the review, found that the existing unsigned verification path was unsafe after removing the equality check. A valid execution could be replayed using different event IDs, bypassing the deduplication check you've set. To handle that then, I changed the integration to require signed Elfa webhooks, passed a secret while creating queries, removed the execution API auth fallback, and added a persisted marker intended to re-arm existing unsigned queries. now this has expanded beyond the original bug fix. It changes the auth architecture, secret management, the query structure and deployment/migration behavior. and introduces more issues Downstream. Existing Elfa queries would need to be recreated, dynamic queries currently do not migrate correctly, and secret rotation could causes all existing notifications to fail verification. [https://github.com/pear-protocol/agent-pear-v2/pull/555](https://github.com/pear-protocol/agent-pear-v2/pull/555 "https://github.com/pear-protocol/agent-pear-v2/pull/555") this is some work I did but it problems more than it solves
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 1:17 PM
    
    seeing what it is
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 1:17 PM
    
    different path ![🥲](https://discord.com/assets/69aa309e02aeb2c4.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 1:06 PM
    
    [https://github.com/pear-protocol/agent-pear-v2/pull/553](https://github.com/pear-protocol/agent-pear-v2/pull/553 "https://github.com/pear-protocol/agent-pear-v2/pull/553") the elfa errors. pls check once
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:46 PM
    
    will keep you posted
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:46 PM
    
    got it thanks.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:39 PM
    
    ahh cool ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:35 PM
    
    ahh okay. is the plan updated and additive or smth fundamental changed?
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:03 PM
    
    22 aug that goes up as well
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:03 PM
    
    plus referall elfa ongoing ![🥶](https://discord.com/assets/726dc886a14ea197.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:02 PM
    
    like last week launch and boom a new venue now
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 12:02 PM
    
    ![😓](https://discord.com/assets/7a079c219a80e127.svg) lighter came outta no where
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:59 AM
    
    platform = lighter support is needed, exactly
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:58 AM
    
    no it does not. on the stats side i did come across support for future venue's but most of the execution and generation parts are hardcoded for HL
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:56 AM
    
    yes the gates are agnostic for venue but the execution and symbol resolution would need to handled in quant db
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:52 AM
    
    yep i mean not in both repos
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:52 AM
    
    its upstream
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:52 AM
    
    not our code fs
    - throws an uncaught error, pear-stat-arbs.onrender.com
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:51 AM
    
    yeah that seemed like transient
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:49 AM
    
    we went quite all out already
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:49 AM
    
    hmm, now it's in motion. debugging it will be sure a task
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:46 AM
    
    true we only have 67 eligible tg users who even received that notification, maybe it fruits in long ![😕](https://discord.com/assets/4b78aed623919cf4.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:45 AM
    
    will share you a doc shortly
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:45 AM
    
    im workin on lighter integration planning
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:44 AM
    
    ![📝](https://discord.com/assets/5f6b1b0353091184.svg) ![😮‍💨](https://discord.com/assets/7986d723975b8636.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:43 AM
    
    im pulling those traces and seeing nw
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:41 AM
    
    these do work, must be smth with those.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:41 AM
    
    ⁠PEAR⁠
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:41 AM
    
    think after the notification people tried to create
    
- ## Prakhar
    
     [PEAR],  _—_ 8/18/2026 11:40 AM
    
    im not sure. it got merged and up around 12 am. i was prepping the notification script and leaderbaord kb update and prompts. just saw these errors as well
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 8:28 PM
    
    lol
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 8:28 PM
    
    just a redeploymnet
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 8:28 PM
    
    works boss
    
    [](https://cdn.discordapp.com/attachments/1519254370397589567/1538924955704496308/image.png?ex=6a95968d&is=6a94450d&hm=019716c94337a2cabd70c6016eb75ad4aa610b531f35f3dc3515b44e639e722a&)
    
    ![Image](https://media.discordapp.net/attachments/1519254370397589567/1538924955704496308/image.png?ex=6a95968d&is=6a94450d&hm=019716c94337a2cabd70c6016eb75ad4aa610b531f35f3dc3515b44e639e722a&=&format=webp&quality=lossless&width=512&height=503)![](data:image/png;base64,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ICH/ISAh/yIgIP8iIB//Ih8e/yIfHf8iHx3/Ih4c/yIeG/8iHRn/Ih0Z/yIcGP8hGxf/IRsW/yAaFf8gGRX/HxkU/x8YFP8eFxP/HRcT/xwWE/8cFhP/GxUT/xoVE/8aFBP/GRQT/xkUE/8YExP/GBMT/xgTE/8AgQB+/wAiISH/IiEh/yIhIf8iISD/IiAg/yMgH/8jIB7/Ix8d/yMfHP8jHxv/Ix4a/yMeGf8jHRj/IxwX/yIcFv8iGxb/IRsV/yEaFP8gGRT/IBkT/x8YE/8eFxP/HRcT/x0WE/8cFhP/GxUT/xsVE/8aFBP/GRQT/xkUE/8ZFBP/GRQT/wCBAH7/ACMhIf8jISH/IyEh/yMhIP8jISD/IyEf/yMgHv8jIB3/IyAc/yMfG/8jHxr/Ix4Z/yMeGP8jHRf/IxwX/yIcFv8iGxX/IRoV/yEaFP8gGRT/HxgT/x8YE/8eFxP/HRcT/xwWE/8cFhP/GxUT/xoVE/8aFRP/GhQT/xkUE/8ZFBP/AIEAfv8AIyIi/yMiIf8jIiH/IyEh/yMhIP8jIR//IyEf/yMgHv8kIB3/JB8c/yQfG/8kHhr/Ix4Z/yMdGP8jHRf/IhwW/yIbFv8hGxX/IRoV/yAaFP8fGRT/HxgU/x4YFP8dFxT/HBcU/xwWFP8bFhT/GxUU/xoVFP8aFRT/GRUU/xkVFP8AgQB+/wAjIiL/IyIi/yMiIv8jIiL/IyEh/yMhIP8jIR//IyEf/yMgHv8jIB3/Ix8c/yMfG/8jHhr/Ix4Z/yMdGP8iHBf/IhwX/yEbFv8hGxb/IBoV/x8ZFf8eGRX/HhgV/x0YFf8cFxX/GxcV/xsWFf8aFhX/GhYV/xkVFf8ZFRX/GRUV/wCBAH7/ACIiJP8iIiT/IiIj/yIiI/8iIiL/IiEi/yIhIf8jISD/IyAf/yMgHv8jIB3/Ix8c/yIfG/8iHhr/Ih0a/yIdGf8hHBj/IBsY/yAbF/8fGhf/HhoX/x4ZF/8dGBb/HBgW/xsXFv8bFxb/GhYX/xoWF/8ZFhf/GRYX/xgVF/8YFRf/AIEAfv8AISIm/yEiJf8hIiX/ISIl/yEiJP8hIiP/ISEj/yIhIv8iISH/IiAg/yIgH/8iHx7/IR8d/yEeHP8hHhz/IB0b/yAcGv8fHBr/HxsZ/x4aGf8dGhn/HRkY/xwZGP8bGBj/GhgY/xoXGP8ZFxj/GRYZ/xgWGf8YFhn/FxYZ/xcWGf8AgQB+/wAgIyj/ICMo/yAiJ/8gIif/ICIm/yAiJv8gIiX/ICEk/yAhI/8gICL/ICAh/yAfIP8gHx//IB4e/yAeHv8fHR3/Hx0c/x4cHP8eGxv/HRsb/xwaG/8bGRv/Gxkb/xoYG/8ZGBv/GBcb/xgXG/8XFxv/FxYb/xYWG/8WFhv/FhYb/wCBAH7/AB4jKv8eIyr/HiIq/x4iKf8eIin/HiIo/x8iJ/8fISb/HyEl/x8gJP8fICT/HyAj/x4fIv8eHiH/Hh4g/x4dH/8dHR//HRwe/xwbHv8bGx7/Gxod/xoaHf8ZGR3/GBgd/xcYHf8XFx3/Fhcd/xYXHf8VFh7/FRYe/xUWHv8UFh7/AIEAfv8AHCMt/xwjLP8cIyz/HCIs/x0iK/8dIir/HSIq/x0hKf8dISj/HSEn/x0gJv8dICX/HR8k/xweJP8cHiP/HB0i/xsdIv8bHCH/Ghsh/xkbIP8ZGiD/GBog/xcZIP8WGSD/Fhgg/xUYIP8UFyD/FBcg/xMXIP8TFiD/ExYg/xMWIf8AgQB+/wAaIy//GiMv/xojL/8bIi7/GyIu/xsiLf8bIiz/GyEs/xshK/8bISr/GyAp/xsgKP8bHyf/Gx8m/xoeJv8aHSX/GR0k/xkcJP8YGyP/GBsj/xcaI/8WGiP/FRkj/xUZI/8UGCP/Exgj/xIXI/8SFyP/ERcj/xEWI/8RFiP/ERYj/wCBAH7/ABgjMv8YIzL/GCIx/xkiMf8ZIjD/GSIw/xkiL/8ZIS7/GSEt/xkhLP8ZICz/GSAr/xkfKv8ZHin/GB4o/xgdKP8XHSf/Fxwn/xYbJv8WGyb/FRom/xQaJv8TGSb/Exkm/xIYJv8RGCb/EBcm/xAXJv8PFyb/DxYm/w8WJv8PFib/AIEAfv8AFiM0/xcjNP8XIjT/FyI0/xciM/8XIjL/FyIy/xchMf8XITD/FyAv/xcgLv8XIC3/Fx8s/xceLP8WHiv/Fh0q/xYdKv8VHCn/FBsp/xQbKf8TGij/Ehoo/xEZKP8RGSj/EBgo/w8YKP8PFyn/Dhcp/w0XKf8NFin/DRYp/w0WKf8AgQB+/wAVIjf/FSI3/xUiNv8VIjb/FSI1/xUiNf8VIjT/FSEz/xUhMv8VIDL/FSAx/xUfMP8VHy//FR4u/xUeLf8UHS3/FB0s/xMcLP8TGyv/Ehsr/xEaK/8QGiv/EBkr/w8ZK/8OGCv/DRgr/w0XK/8MFyv/DBcr/wsWK/8LFiz/CxYs/wCBAH7/ABMiOf8TIjn/EyI5/xMiOP8TIjj/EyI3/xQhNv8UITX/FCE1/xQgNP8UIDP/FB8y/xMfMf8THjD/Ex4w/xMdL/8SHS7/ERwu/xEbLv8QGy3/Dxot/w8aLf8OGS3/DRkt/wwYLf8MGC3/Cxct/wsXLv8KFy7/ChYu/wkWLv8JFi7/AIEAfv8AEiI7/xIiO/8SIjr/EiI6/xIiOf8SIjn/EiE4/xIhN/8SITb/EiA2/xIgNf8SHzT/Eh8z/xIeMv8SHjL/ER0x/xEdMP8QHDD/Dxsw/w8bL/8OGi//DRov/w0ZL/8MGC//Cxgv/woYL/8KFy//CRcv/wkXMP8IFjD/CBYw/wgWMP8AgQB+/wARIjz/ESI8/xEiPP8RIjv/ESI7/xEiOv8RITr/ESE5/xEhOP8RIDf/ESA2/xEfNf8RHzT/ER40/xAeM/8QHTL/EB0y/w8cMf8OGzH/Dhsx/w0aMf8MGjD/Cxkw/wsYMP8KGDH/CRgx/wkXMf8IFzH/CBcx/wcWMf8HFjH/BxYx/wCBAH7/ABAiPf8QIj3/ECI9/xAiPP8QIjz/ECI7/xAhO/8QITr/ECE5/xAgOP8QIDf/EB82/xAfNf8QHjX/EB40/w8dM/8PHDP/Dhwy/w4bMv8NGzL/DBoy/wsZMf8LGTH/Chgy/wkYMv8IGDL/CBcy/wcXMv8HFzL/BhYy/wYWMv8GFjL/AYEAfv8ADyI+/w8iPv8PIj3/ECI9/xAiPP8QIjz/ECE7/xAhOv8QITn/ECA5/xAgOP8QHzf/EB82/xAeNf8PHjT/Dx00/w4cM/8OHDP/DRsz/w0bMv8MGjL/Cxky/woZMv8KGDL/CRgy/wgXMv8HFzL/Bxcy/wYXM/8GFjP/BhYz/wYWM//x/Uo6bZvW8wAAAABJRU5ErkJggg==)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 7:54 PM
    
    railway chart-image-generator
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 7:54 PM
    
    before going could you just restart / redeploy it? some process might be wedged.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 7:47 PM
    
    cool okay
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 6:28 PM
    
    yeah ill prep the promotion pr, conflicts are there have to see them. then get to it
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 6:26 PM
    
    ill start and keep working. will be in loop and ask things. no prblm
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 6:25 PM
    
    i got confused for a sec there when said baskets not generating lol
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 6:24 PM
    
    new thing
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 6:24 PM
    
    yeahh
    
- ## Prakhar
    
     [PEAR],  _—_ 8/17/2026 6:02 PM
    
    yes i searched around boundaries and cases with multiple wallet and multiple account. then this surfaced.
    - no no that i checked. its on latest
    
-  [PEAR], 
    
    ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 8:44 PM
    
    one thing i did notice, it's mostly around tg's update. last time it was also the next day i saw this appear more and even yesterday or a day earlier tg got update
    
- ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 8:43 PM
    
    yes, it’s the streamed text part. the BTC chart is from the previous turn. and it's mostly transient or related to tg's end. jeyd and i have come across this earlier on the desktop app too. i tried reproducing it but honestly haven't been able to trigger it myself yet across many many msg or tries, though people do seem to hit it occasionally.
    
-  [PEAR], 
    
    ## Prakhar
    
     [PEAR],  _—_ 8/28/2026 7:42 PM
    
    checked your session ser, same prblm. the agent is emitting the response correctly, but it isn't being streamed/rendered properly on Telegram Desktop. i looked into ways to prevent this from our side, but atp the cost of the "fix" seems higher than the benefit, and the fixes aren't fully guaranteed either since this is from TG's side i've added more logging for now to pinpoint exactly where this delivery gets affected. i'll share more once i have something solid. Telegram Desktop also got an update yesterday, so it could be related to that. pls update as well in case not.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/27/2026 6:31 PM
    
    Yes yes. I was looking into these today, will share complete findings asap
    
- ## Prakhar
    
     [PEAR],  _—_ 8/27/2026 6:28 PM
    
    Yes ![😓](https://discord.com/assets/7a079c219a80e127.svg)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/27/2026 6:27 PM
    
    Yes ser. ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg) i did flag the categories failure earlier ago, but hadn’t connected it to the stale DB data here. checked the paths now and we don’t have proper freshness alerts, mostly just success messages/internal timestamps. should be fairly simple to add a stale data check so we know when these datasets stop refreshing.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 8:36 PM
    
    making it venue aware now
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 3:59 PM
    
    ah ok ok
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 3:57 PM
    
    my bad i checked further stats are unable to be polled yet, lacks historical data. it fallsback to market snapshots or HL references cc @jeyd
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1541393620865650728/image.png?ex=6a95572c&is=6a9405ac&hm=eb33b9cae285ec3e6c8156c862d066f40bf80cfdf04a4c8d13cfb6fa2cf0d09f&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1541393620865650728/image.png?ex=6a95572c&is=6a9405ac&hm=eb33b9cae285ec3e6c8156c862d066f40bf80cfdf04a4c8d13cfb6fa2cf0d09f&=&format=webp&quality=lossless&width=420&height=512)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABsAAAAgCAYAAADjaQM7AAAORklEQVR4AQBtAJL/ACg0Qv8oNEL/KDRC/yg0Qv8oNEL/KDRC/yczQv8nM0H/JzNB/yYzQf8mMkD/JjJA/yUxQP8lMT//JTE//yQwPv8kMD7/IzA+/yMvPf8jLz3/Iy89/yIuPf8iLjz/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACg0Qv8oNEL/KDRC/yg0Qv8oNEL/JzRC/yczQv8nM0H/JzNB/yYyQf8mMkD/JjJA/yUxQP8lMT//JTE//yQwPv8kMD7/IzA+/yMvPf8jLz3/Iy89/yIuPf8iLjz/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACg0Qv8oNEL/KDRC/yg0Qv8oNEL/JzRC/yczQf8nM0H/JzNB/yYyQf8mMkD/JjJA/yUxQP8lMT//JDE//yQwPv8kMD7/IzA+/yMvPf8jLz3/Iy89/yIuPf8iLjz/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACg0Qv8oNEL/KDRC/yg0Qv8oNEL/JzNC/yczQf8nM0H/JzNB/yYyQf8mMkD/JjJA/yUxP/8lMT//JDE//yQwPv8kMD7/IzA+/yMvPf8jLz3/Iy89/yIuPf8iLjz/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACg0Qv8oNEL/KDRC/yg0Qv8nNEL/JzNB/yczQf8nM0H/JjNB/yYyQP8mMkD/JTJA/yUxP/8lMT//JDE//yQwPv8kMD7/IzA+/yMvPf8jLz3/Iy89/yIuPf8iLjz/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACg0Qv8oNEL/JzRC/yczQv8nM0H/JzNB/yczQf8nM0H/JjJB/yYyQP8mMkD/JTFA/yUxP/8lMT//JDA//yQwPv8kMD7/IzA+/yMvPf8jLz3/Iy89/yIuPf8iLjz/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACczQv8nM0L/JzNC/yczQf8nM0H/JzNB/yczQf8mMkH/JjJA/yYyQP8mMkD/JTFA/yUxP/8lMT//JDA//yQwPv8kMD7/Iy8+/yMvPf8jLz3/Iy89/yIvPf8iLj3/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACczQf8nM0H/JzNB/yczQf8nM0H/JzNB/yYyQf8mMkD/JjJA/yYyQP8lMUD/JTE//yUxP/8kMT//JDA+/yQwPv8kMD7/Iy8+/yMvPf8jLz3/Iy89/yIvPf8iLj3/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACczQf8nM0H/JzNB/yczQf8mM0H/JjJB/yYyQP8mMkD/JjJA/yUyQP8lMT//JTE//yUxP/8kMT//JDA+/yQwPv8kMD7/Iy8+/yMvPf8jLz3/Iy89/yIvPf8iLj3/Ii48/yIuPP8iLjz/Ii48/wBtAJL/ACYzQf8mM0H/JjJB/yYyQf8mMkD/JjJA/yYyQP8mMkD/JTJA/yUxQP8lMT//JTE//yQxP/8kMD//JDA+/yQwPv8jMD7/Iy8+/yMvPf8jLz3/Iy89/yIvPf8iLj3/Ii49/yIuPP8iLjz/Ii48/wBtAJL/ACYyQP8mMkD/JjJA/yYyQP8mMkD/JjJA/yYyQP8lMkD/JTE//yUxP/8lMT//JTE//yQwP/8kMD7/JDA+/yQwPv8jMD7/Iy8+/yMvPf8jLz3/Iy89/yMvPf8iLz3/Ii49/yIuPf8iLjz/Ii48/wBtAJL/ACYyQP8mMkD/JjJA/yYyQP8lMkD/JTFA/yUxQP8lMT//JTE//yUxP/8lMT//JDA//yQwPv8kMD7/JDA+/yQwPv8jMD7/Iy89/yMvPf8jLz3/Iy89/yMvPf8iLz3/Ii89/yIuPf8iLj3/Ii49/wBtAJL/ACUxQP8lMUD/JTFA/yUxP/8lMT//JTE//yUxP/8lMT//JTE//yQxP/8kMD//JDA+/yQwPv8kMD7/JDA+/yMwPv8jLz7/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Ii89/yIvPf8iLz3/Ii49/wBtAJL/ACUxP/8lMT//JTE//yUxP/8lMT//JTE//yUxP/8kMT//JDA//yQwP/8kMD7/JDA+/yQwPv8kMD7/JDA+/yMwPv8jLz7/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yIvPf8iLz3/Ii89/wBtAJL/ACQxP/8kMT//JDE//yQxP/8kMD//JDA//yQwPv8kMD7/JDA+/yQwPv8kMD7/JDA+/yQwPv8jMD7/IzA+/yMvPv8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACQwPv8kMD7/JDA+/yQwPv8kMD7/JDA+/yQwPv8kMD7/JDA+/yQwPv8kMD7/IzA+/yMwPv8jLz7/Iy8+/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACQwPv8kMD7/JDA+/yQwPv8kMD7/JDA+/yMwPv8jMD7/IzA+/yMwPv8jLz7/Iy8+/yMvPv8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACIuPf8iLj3/Ii49/yIuPf8iLz3/Ii89/yIvPf8iLz3/Ii89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACIuPP8iLjz/Ii48/yIuPP8iLjz/Ii48/yIuPP8iLjz/Ii49/yIuPf8iLj3/Ii89/yIvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/wBtAJL/ACIuPP8iLjz/Ii48/yIuPP8iLjz/Ii48/yIuPP8iLjz/Ii48/yIuPP8iLjz/Ii49/yIuPf8iLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy8+/yMvPv8jLz7/Iy8+/wBtAJL/ACEtPP8hLTz/IS08/yEtPP8hLTz/IS48/yEuPP8iLjz/Ii48/yIuPP8iLjz/Ii48/yIuPP8iLj3/Ii89/yMvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz7/Iy8+/yMvPv8jLz7/Iy8+/wBtAJL/ACEtO/8hLTv/IS07/yEtO/8hLTv/IS07/yEtPP8hLTz/IS48/yIuPP8iLjz/Ii48/yIuPP8iLjz/Ii49/yIvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPf8jLz7/Iy8+/yMwPv8jMD7/IzA+/wBtAJL/ACEtO/8hLTv/IS07/yEtO/8hLTv/IS07/yEtO/8hLTv/IS08/yEuPP8iLjz/Ii48/yIuPP8iLjz/Ii49/yIvPf8jLz3/Iy89/yMvPf8jLz3/Iy89/yMvPv8jLz7/IzA+/yMwPv8jMD7/JDA+/wBtAJL/ACAsO/8gLDv/ICw7/yAtO/8hLTv/IS07/yEtO/8hLTv/IS07/yEtO/8hLjz/Ii48/yIuPP8iLjz/Ii48/yIuPf8iLz3/Iy89/yMvPf8jLz3/Iy89/yMvPv8jMD7/IzA+/yQwPv8kMD7/JDA+/wBtAJL/ACAsOv8gLDr/ICw6/yAsOv8gLDv/IC07/yEtO/8hLTv/IS07/yEtO/8hLTz/IS48/yIuPP8iLjz/Ii48/yIuPf8iLz3/Iy89/yMvPf8jLz3/Iy89/yMvPv8jMD7/IzA+/yQwPv8kMD7/JDA+/wBtAJL/ACAsOv8gLDr/ICw6/yAsOv8gLDr/ICw7/yAtO/8hLTv/IS07/yEtO/8hLTv/IS08/yIuPP8iLjz/Ii48/yIuPf8iLz3/Iy89/yMvPf8jLz3/Iy89/yMvPv8jMD7/JDA+/yQwPv8kMD7/JDA+/wBtAJL/ACAsOv8gLDr/ICw6/yAsOv8gLDr/ICw6/yAsO/8gLTv/IS07/yEtO/8hLTv/IS08/yEuPP8iLjz/Ii48/yIuPP8iLz3/Iy89/yMvPf8jLz3/Iy89/yMvPv8jMD7/JDA+/yQwPv8kMD7/JDA+/wBtAJL/ACAsOv8gLDr/ICw6/yAsOv8gLDr/ICw6/yAsOv8gLDv/IC07/yEtO/8hLTv/IS07/yEuPP8iLjz/Ii48/yIuPP8iLz3/Iy89/yMvPf8jLz3/Iy8+/yMvPv8jMD7/JDA+/yQwPv8kMD7/JDA+/wBtAJL/AB8sOv8fLDr/ICw6/yAsOv8gLDr/ICw6/yAsOv8gLDv/IC07/yEtO/8hLTv/IS07/yEuPP8iLjz/Ii48/yIuPP8iLz3/Iy89/yMvPf8jLz3/Iy8+/yMwPv8kMD7/JDA+/yQwPv8kMD7/JDA+/wFtAJL/AB8sOv8fLDr/ICw6/yAsOv8gLDr/ICw6/yAsOv8gLDv/IC07/yEtO/8hLTv/IS07/yEtPP8iLjz/Ii48/yIuPP8iLz3/Iy89/yMvPf8jLz3/Iy8+/yMwPv8kMD7/JDA+/yQwPv8kMD7/JDA+/59HQ6WIYOMHAAAAAElFTkSuQmCC)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 3:49 PM
    
    the baskets are up and working, i have to add some integration tests now
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1541391455313068074/image.png?ex=6a955528&is=6a9403a8&hm=e93ed0ffb36f6f120ae125087063b18bac76b47ea217bcf3f1cd7564eab7712a&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1541391455313068074/image.png?ex=6a955528&is=6a9403a8&hm=e93ed0ffb36f6f120ae125087063b18bac76b47ea217bcf3f1cd7564eab7712a&=&format=webp&quality=lossless&width=355&height=512)![](data:image/png;base64,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)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 3:39 PM
    
    yes it was wip, def makes better sense. has been updated.
    

- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 2:55 PM
    
    gm bro. this is now in dev, will be shipped with next dev to prod merge [https://github.com/pear-protocol/agent-pear-v2/pull/569](https://github.com/pear-protocol/agent-pear-v2/pull/569 "https://github.com/pear-protocol/agent-pear-v2/pull/569")
    

- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 2:55 PM
    
    bullish ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg) updated and getting it to dev now
    

- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 1:31 PM
    
    ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg) on it ser
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 1:23 PM
    
    checked this. yes, since INJ doesent have 10x, upto 5x, so inorder to preserve the maths it logically doubled the amount at stake which is by code correct but visually not consistent or expressive that it is doing that. updating the agent response to inform the person about it more clearly beforehand @TP.hl @Abdulmuizz
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 1:09 PM
    
    ![:pepenoted:](https://cdn.discordapp.com/emojis/1176912072316035122.webp?size=56) checking
    

- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 9:21 AM
    
    gm boss @huf 1 - this wording has been there for HL prior as well. have noted and will update 2 - missing update in docs, pr raised. would get it merged post review. 3 - had to be updated post pair and basket work in stats repo. done. wired the config now as well. thanks ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg) 4- think @FE could help out here. for link wallet flow, mentions HL as of now [https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/routes/connect/telegram.tsx](https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/routes/connect/telegram.tsx "https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/routes/connect/telegram.tsx") trade panel [https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/trade-left-panel-shell.tsx#L2](https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/trade-left-panel-shell.tsx#L2 "https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/trade-left-panel-shell.tsx#L2") Gate Check [https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/hl-trade-footer.tsx#L44-L51](https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/hl-trade-footer.tsx#L44-L51 "https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/hl-trade-footer.tsx#L44-L51") CTA [https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/hl-trade-footer.tsx#L99-L132](https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/hl-trade-footer.tsx#L99-L132 "https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/components/trade/components/trade-left-panel/hl-trade-footer.tsx#L99-L132") helper [https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/lib/pear-sdk/clients.ts#L490-L498](https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/lib/pear-sdk/clients.ts#L490-L498 "https://github.com/pear-protocol/frontend/blob/cdaf58315aba5d1fba1a507f73af5e53080350df/apps/pear-web/src/lib/pear-sdk/clients.ts#L490-L498") this is there but isnt being used by UI and some places more. 5 - so new wallet picks Lighter , footer runs HL check, and offers HL setup. Lighter path never triggered
    
- ## Prakhar
    
     [PEAR],  _—_ 8/24/2026 9:21 AM
    
    gm boss @Caroulemapoule  
    the Lighter integrated AP is on development, [https://t.me/agent_pear_dev_bot](https://t.me/agent_pear_dev_bot "https://t.me/agent_pear_dev_bot") , please try it here. that should do. it isnt in prod yet and the agent pear web is accessible here at [https://lighter.pear.garden/agent-pear](https://lighter.pear.garden/agent-pear "https://lighter.pear.garden/agent-pear")
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1541293903204851842/image.png?ex=6a95a30e&is=6a94518e&hm=7b8133863e8fd1de7c5fb7aae2f627651dda114865da56dfb294aa11bea33b19&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1541293903204851842/image.png?ex=6a95a30e&is=6a94518e&hm=7b8133863e8fd1de7c5fb7aae2f627651dda114865da56dfb294aa11bea33b19&=&format=webp&quality=lossless&width=320&height=247)![](data:image/png;base64,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)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1541293903762821191/image.png?ex=6a95a30e&is=6a94518e&hm=0f4088d3717490fcc577a64f9736dea7e44b9d3a492b0245165d8b1618aa3f9d&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1541293903762821191/image.png?ex=6a95a30e&is=6a94518e&hm=0f4088d3717490fcc577a64f9736dea7e44b9d3a492b0245165d8b1618aa3f9d&=&format=webp&quality=lossless&width=320&height=320)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAXCAYAAABqBU3hAAAMEElEQVR4AQCBAH7/AB0dGf8dHRn/HR0Z/x4dGf8eHRn/HhwZ/x4cGf8eHBn/HxwZ/x8bGf8fGxn/HxsZ/x8bGf8gGxn/IBsZ/yAbGf8gGxn/HxsZ/x8bGf8fGxn/HxwZ/x8cGf8eHBn/HhwZ/x4dGf8dHRn/HR0Z/x0eGf8cHhn/HB4Z/xweGf8cHhn/AIEAfv8AHR0Z/x0dGf8dHRn/Hh0Z/x4dGf8eHBn/HhwZ/x4cGf8fHBn/HxsZ/x8bGf8fGxn/IBsZ/yAbGf8gGxn/IBsZ/yAbGf8fGxn/HxsZ/x8bGf8fHBn/HxwZ/x4cGf8eHBn/Hh0Z/x0dGf8dHRn/HR4Z/xweGf8cHhn/HB4Z/xweGf8AgQB+/wAdHRn/HR0Z/x0dGf8eHRn/Hh0Z/x4cGf8eHBn/HxwZ/x8cGf8fGxn/HxsZ/x8bGf8gGxn/IBsZ/yAbGf8gGxn/IBsZ/yAbGf8fGxn/HxsZ/x8bGf8fHBn/HhwZ/x4cGf8eHRn/HR0Z/x0dGf8dHhn/HB4Z/xweGf8cHhn/HB4Z/wCBAH7/AB0dGf8dHRn/Hh0Z/x4dGf8eHBn/HhwZ/x4cGf8fHBn/HxwZ/x8bGf8fGxn/HxsZ/yAbGf8gGxn/IBsZ/yAbGf8gGxn/IBsZ/x8bGf8fGxn/HxsZ/x8cGf8eHBn/HhwZ/x4dGf8dHRn/HR0Z/x0eGf8dHhn/HB4Z/xweGf8cHhn/AIEAfv8AHR0Z/x4dGf8eHRn/Hh0Z/x4cGf8eHBn/HhwZ/x8cGf8fGxn/HxsZ/x8bGf8gGxn/IBsZ/yAbGf8gGxn/IBsZ/yAbGf8gGxn/HxsZ/x8bGf8fGxn/HxwZ/x4cGf8eHBn/Hh0Z/x0dGf8dHRn/HR4Z/x0eGf8cHhn/HB4Z/xweGf8AgQB+/wAeHRn/Hh0Z/x4dGf8eHRn/HhwZ/x4cGf8eHBn/HxwZ/x8bGf8fGxn/HxsZ/yAbGf8gGxn/IBsZ/yAbGf8gGxn/IBsZ/yAbGf8fGxn/HxsZ/x8bGf8fHBn/HhwZ/x4cGf8eHRn/HR0Z/x0dGf8dHhn/HR4Z/xweGf8cHhn/HB4Z/wCBAH7/AB0dGf8eHRn/Hh0Z/x4dGf8eHBn/HhwZ/x4cGf8fHBn/HxsZ/x8bGf8fGxn/IBsZ/yAbGf8gGxn/IBsZ/yAbGf8gGxn/IBsZ/x8bGf8fGxn/HxsZ/x8cGf8eHBn/HhwZ/x4dGf8dHRn/HR0Z/x0eGf8cHhn/HB4Z/xweGf8cHhn/AIEAfv8AHR0Z/x0dGf8eHRn/Hh0Z/x4cGf8eHBn/HhwZ/x8cGf8fHBn/HxsZ/x8bGf8fGxn/IBsZ/yAbGf8gGxn/IBsZ/yAbGf8fGxn/HxsZ/x8bGf8fHBn/HhwZ/x4cGf8eHRn/HR0Z/x0dGf8dHhn/HR4Z/xweGf8cHhn/HB4Z/xweGf8AgQB+/wAdHRn/HR0Z/x0dGf8eHRn/Hh0Z/x4cGf8eHBn/HhwZ/x8cGf8fHBn/HxsZ/x8bGf8fGxn/HxsZ/x8bGf8fGxn/HxsZ/x8bGf8fGxn/HxwZ/x8cGf8eHBn/HhwZ/x4dGf8dHRn/HR0Z/x0eGf8cHhn/HB4Z/xweGf8cHxn/HB8Z/wCBAH7/AB0dGf8dHRn/HR0Z/x0dGf8dHRn/Hh0Z/x4dGf8eHBn/HhwZ/x8cGf8fHBn/HxsZ/x8bGf8fGxn/HxsZ/x8bGf8fGxn/HxsZ/x8cGf8fHBn/HhwZ/x4cGf8eHRn/HR0Z/x0dGf8dHhn/HB4Z/xweGf8cHxn/HB8Z/xsfGf8bHxn/AIEAfv8AHR4Z/x0eGf8dHhn/HR4Z/x0dGf8dHRn/HR0Z/x4dGf8eHBn/HhwZ/x4cGf8eHBn/HxwZ/x8cGf8fHBn/HxwZ/x8cGf8eHBn/HhwZ/x4cGf8eHRn/HR0Z/x0dGf8dHhn/HB4Z/xweGf8cHxn/Gx8Z/xsfGf8bHxn/Gx8Z/xsfGf8AgQB+/wAcHhn/HB4Z/xweGf8cHhn/HR4Z/x0eGf8dHRn/HR0Z/x0dGf8eHRn/Hh0Z/x4cGf8eHBn/HhwZ/x4cGf8eHBn/HhwZ/x4cGf8eHRn/HR0Z/x0dGf8dHRn/HR4Z/xweGf8cHxn/Gx8Z/xsfGf8bHxn/GyAZ/xogGf8aIBn/GiAZ/wCBAH7/ABsfGf8bHxn/HB8Z/xwfGf8cHhn/HB4Z/xweGf8dHhn/HR4Z/x0dGf8dHRn/HR0Z/x0dGf8eHRn/Hh0Z/x4dGf8dHRn/HR0Z/x0dGf8dHhn/HR4Z/xweGf8cHxn/HB8Z/xsfGf8bIBn/GiAZ/xogGf8aIBn/GiEZ/xohGf8aIRn/AIEAfv8AGyAZ/xsgGf8bIBn/Gx8Z/xsfGf8bHxn/HB8Z/xwfGf8cHhn/HB4Z/xweGf8dHhn/HR4Z/x0eGf8dHhn/HR4Z/x0eGf8dHhn/HB4Z/xweGf8cHxn/Gx8Z/xsfGf8bIBn/GiAZ/xogGf8aIRn/GSEZ/xkhGf8ZIRn/GSIZ/xkiGf8AgQB+/wAaIBn/GiAZ/xogGf8aIBn/GiAZ/xsgGf8bIBn/Gx8Z/xsfGf8bHxn/HB8Z/xwfGf8cHhn/HB4Z/xweGf8cHhn/HB8Z/xwfGf8bHxn/Gx8Z/xsfGf8bIBn/GiAZ/xogGf8aIRn/GSEZ/xkiGf8ZIhn/GCIZ/xgiGf8YIhn/GCIZ/wCBAH7/ABkhGf8ZIRn/GSEZ/xkhGf8aIRn/GiEZ/xogGf8aIBn/GiAZ/xsgGf8bIBn/Gx8Z/xsfGf8bHxn/Gx8Z/xsfGf8bHxn/GyAZ/xsgGf8aIBn/GiAZ/xohGf8ZIRn/GSEZ/xkiGf8YIhn/GCIZ/xgjGf8XIxn/FyMZ/xcjGf8XIxn/AIEAfv8AGCIZ/xgiGf8YIhn/GSIZ/xkiGf8ZIRn/GSEZ/xkhGf8aIRn/GiEZ/xogGf8aIBn/GiAZ/xogGf8aIBn/GiAZ/xogGf8aIBn/GiEZ/xkhGf8ZIRn/GSIZ/xgiGf8YIhn/GCMZ/xcjGf8XIxn/FyQZ/xYkGf8WJBn/FiQZ/xYkGf8AgQB+/wAXIxn/FyMZ/xgjGf8YIxn/GCMZ/xgiGf8YIhn/GCIZ/xkiGf8ZIRn/GSEZ/xkhGf8ZIRn/GSEZ/xkhGf8ZIRn/GSEZ/xkhGf8ZIhn/GSIZ/xgiGf8YIhn/GCMZ/xcjGf8XJBn/FiQZ/xYkGf8WJRn/FiUZ/xUlGf8VJRn/FSUZ/wCBAH7/ABckGf8XJBn/FyQZ/xcjGf8XIxn/FyMZ/xgjGf8YIxn/GCIZ/xgiGf8YIhn/GCIZ/xkiGf8ZIhn/GSIZ/xkiGf8YIhn/GCIZ/xgiGf8YIxn/FyMZ/xcjGf8XJBn/FiQZ/xYkGf8WJRn/FSUZ/xUlGf8VJhn/FSYZ/xQmGf8UJhn/AIEAfv8AFiQZ/xYkGf8WJBn/FiQZ/xYkGf8XJBn/FyQZ/xcjGf8XIxn/FyMZ/xgjGf8YIxn/GCIZ/xgiGf8YIhn/GCIZ/xgjGf8YIxn/FyMZ/xcjGf8XJBn/FiQZ/xYkGf8WJRn/FSUZ/xUlGf8VJhn/FCYZ/xQmGf8UJhn/FCcZ/xQnGf8AgQB+/wAWJRn/FiUZ/xYlGf8WJRn/FiQZ/xYkGf8WJBn/FyQZ/xckGf8XIxn/FyMZ/xcjGf8XIxn/FyMZ/xcjGf8XIxn/FyMZ/xcjGf8XJBn/FyQZ/xYkGf8WJBn/FiUZ/xUlGf8VJhn/FCYZ/xQmGf8UJxn/FCcZ/xMnGf8TJxn/EycZ/wCBAH7/ABUlGf8VJRn/FSUZ/xUlGf8WJRn/FiUZ/xYkGf8WJBn/FiQZ/xckGf8XJBn/FyMZ/xcjGf8XIxn/FyMZ/xcjGf8XJBn/FyQZ/xYkGf8WJBn/FiQZ/xYlGf8VJRn/FSYZ/xQmGf8UJhn/FCcZ/xMnGf8TJxn/EycZ/xMoGf8TKBn/AYEAfv8AFSUZ/xUlGf8VJRn/FSUZ/xUlGf8WJRn/FiUZ/xYkGf8WJBn/FiQZ/xckGf8XJBn/FyQZ/xckGf8XJBn/FyQZ/xckGf8WJBn/FiQZ/xYkGf8WJRn/FSUZ/xUlGf8VJhn/FCYZ/xQnGf8UJxn/EycZ/xMnGf8TKBn/EygZ/xMoGf9CWM0IZf2rbwAAAABJRU5ErkJggg==)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 2:26 PM
    
    Ah its the same as Hl, first navigate to pro.pear.garden There in the accounts menu , click add trade account and select venue as lighter. Once that is set up , please resync and it should be up and running.
    
- ## Prakhar
    
     [PEAR],  _—_ 8/21/2026 3:11 AM
    
    gm @huf @Caroulemapoule @jeyd AP + Lighter Integration is live on dev. [https://t.me/agent_pear_dev_bot](https://t.me/agent_pear_dev_bot "https://t.me/agent_pear_dev_bot") Initial testing is done across common requests and edge cases, will be testing it today more, please share your findings and suggestions as well. Feature additions:
    
    - Lighter account linking
    - Multi venue support for Hyperliquid and Lighter, Extensible to other platforms
    - Venue aware account selection for trades and portfolio reads
    - Lighter market data, asset validation, sizing, and leverage checks
    
    cc - @prudhvi ![🫡](https://discord.com/assets/753b4e9a8f4ddd54.svg)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1540113636931145728/image.png?ex=6a954c58&is=6a93fad8&hm=5e908af7a6b370e35c77e839bb8ed0f0251cd3acdaddba455005837ea1979edf&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1540113636931145728/image.png?ex=6a954c58&is=6a93fad8&hm=5e908af7a6b370e35c77e839bb8ed0f0251cd3acdaddba455005837ea1979edf&=&format=webp&quality=lossless&width=320&height=267)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAASCAYAAAA6yNxSAAAJcklEQVR4AQCBAH7/ACI3T/8iN0//IjdQ/yE3Uf8gNlL/HzZT/x42Vf8dNVf/HDVY/xo0Wv8ZNFz/GDNd/xczX/8WM2D/FjJh/xUyYv8VMmL/FDJi/xUyYv8VMmL/FTJh/xYzYf8WM2D/FzNf/xgzXv8YNF3/GTRc/xo0W/8aNVr/GzVZ/xs1Wf8bNVn/AIEAfv8AIzdO/yM3Tv8iN0//IjdQ/yE2Uf8gNlL/HzVU/x41Vv8cNFf/GzRZ/xozW/8ZM1z/GDNe/xcyX/8WMmD/FjJh/xUyYf8VMmH/FTJh/xUyYf8WMmD/FjJg/xcyX/8YM17/GDNd/xkzW/8aNFr/GjRZ/xs0Wf8bNFj/HDVY/xw1V/8AgQB+/wAkN0z/JDZN/yQ2Tf8jNk7/IjZP/yE1Uf8gNVL/HzRU/x40Vf8cM1f/GzNZ/xoyWv8ZMlz/GDJd/xcxXv8XMV//FjFf/xYxX/8WMV//FzFf/xcxXv8XMl3/GDJc/xkyW/8ZM1r/GjNZ/xszWP8cM1f/HDRW/x00Vv8dNFX/HTRV/wCBAH7/ACY2Sv8mNkr/JTZL/yQ1S/8kNU3/IzVO/yI0T/8gNFH/HzNT/x4zVP8dMlb/HDJY/xsxWf8aMVr/GTFb/xgwXP8YMFz/GDBc/xgwXP8YMFz/GDFb/xkxWv8aMVn/GjFY/xsyV/8cMlb/HDJV/x0zVP8eM1P/HjNS/x4zUv8fM1L/AIEAfv8AKDVH/yc1R/8nNUf/JjRI/yU0Sf8lNEv/IzNM/yIzTv8hMk//IDJR/x8xU/8dMVT/HDBV/xswV/8bMFf/Gi9Y/xovWP8aL1n/Gi9Y/xovWP8aMFf/GzBW/xswVf8cMFT/HTFT/x4xUv8eMVH/HzJQ/yAyT/8gMk7/IDJO/yAyTv8AgQB+/wAqNEP/KTRD/yk0RP8oM0X/JzNG/yYzR/8lMkn/JDJK/yMxTP8iMU3/IDBP/x8wUP8eL1L/HS9T/x0vVP8cLlT/HC5U/xwuVP8cLlT/HC5U/xwuU/8dL1L/HS9R/x4vUP8fL0//HzBO/yAwTP8hMEv/ITFK/yIxSv8iMUn/IjFJ/wCBAH7/ACszP/8rM0D/KzNA/yoyQf8pMkL/KDFD/ycxRf8mMUb/JTBI/yMvSv8iL0v/IS5M/yAuTv8fLk//Hi1P/x4tUP8dLVD/HS1Q/x0tUP8dLU//Hi1O/x4tTv8fLkz/IC5L/yAuSv8hLkn/Ii9I/yMvR/8jL0b/JC9F/yQvRP8kMET/AIEAfv8ALTI8/ywyPP8sMT3/KzE+/ysxP/8qMED/KDBB/ycvQ/8mL0T/JS5G/yQuR/8iLUj/IS1K/yAsS/8gLEv/HyxM/x8sTP8fLEz/HyxL/x8sS/8fLEr/ICxJ/yAsSP8hLEf/Ii1F/yMtRP8jLUP/JC5C/yQuQf8lLkD/JS5A/yUuP/8AgQB+/wAuMTn/LTA5/y0wOf8sMDr/LDA7/ysvPf8pLz7/KC4//ycuQf8mLUL/JC1E/yMsRf8iLEb/IStH/yErR/8gK0j/ICpI/yAqSP8gKkf/ICpH/yAqRv8hK0X/IStE/yIrQv8jK0H/IyxA/yQsPv8lLD3/JSw8/yYtPP8mLTv/Ji07/wCBAH7/AC4wNv8uLzb/LS83/y0vN/8sLzj/Ky46/youO/8pLTz/Jy0+/yYsP/8lK0D/JCtC/yMqQ/8iKkT/ISpE/yApRP8gKUT/IClE/yApRP8gKUP/ISlC/yEpQf8iKkD/Iio+/yMqPf8kKjz/JCs6/yUrOf8mKzj/Jis3/yYrN/8nKzf/AIEAfv8ALi80/y4uNP8tLjT/LS41/ywuNv8rLTf/Ki05/ygsOv8nLDv/Jis9/yUqPv8kKj//IylA/yIpQf8hKUH/IChB/yAoQf8gKEH/IChB/yAoQP8gKD//ISg+/yEoPP8iKTv/Iyk5/yQpOP8kKTf/JSo2/yUqNf8mKjT/Jioz/yYqM/8AgQB+/wAtLjL/LS4y/y0tM/8sLTP/Ky00/yosNf8pLDf/KCs4/ycrOf8lKjv/JCo8/yMpPf8iKT7/ISg+/yAoP/8gJz//Hyc//x8nP/8fJz7/Hyc9/yAnPP8gJzv/ISc5/yEoOP8iKDf/Iyg1/yQoNP8kKTP/JSky/yUpMf8mKTD/Jikw/wCBAH7/ACwtMf8sLTH/LC0x/yssMv8qLDP/KSw0/ygrNf8nKzf/JSo4/yQpOf8jKTr/Iig7/yEoPP8gJz3/Hyc9/x8nPf8eJj3/HiY9/x4mPP8eJjv/HyY6/x8mOf8gJzf/ICc2/yEnNP8iJzP/Iigx/yMoMP8kKC//JCgu/yQoLv8lKC3/AIEAfv8AKyww/yssMP8qLDH/Kiwx/ykrMv8oKzP/Jys0/yUqNv8kKTf/Iyk4/yIoOf8hKDr/Hyc7/x8nPP8eJjz/HSY8/x0mPP8dJjv/HSU7/x0lOv8dJjj/HiY3/x4mNv8fJjT/ICYy/yAnMf8hJzD/Iicu/yInLf8jJyz/Iycs/yMoLP8AgQB+/wAqLC//KSww/yksMP8oKzH/Jysy/yYrM/8lKjT/JCo1/yMpNv8iKDf/ICg5/x8nOf8eJzr/HSY7/xwmO/8cJTv/HCU7/xslOv8bJTr/HCU4/xwlN/8cJTb/HSU0/x4lM/8eJjH/HyYw/yAmLv8gJi3/IScs/yEnK/8iJyr/Iicq/wCBAH7/ACgsL/8oLC//KCsw/ycrMf8mKzH/JSoy/yQqNP8jKTX/Iik2/yAoN/8fJzj/Hic5/x0mOv8cJjr/GyU7/xslO/8aJTr/GiU6/xolOf8aJTj/GyU3/xslNf8cJTT/HCUy/x0lMP8eJi//HyYt/x8mLP8gJiv/ICYq/yEmKv8hJin/AIEAfv8AKCwv/ycrL/8nKzD/Jisw/yUrMf8kKjL/Iyoz/yIpNf8hKDb/Hyg3/x4nOP8dJzn/HCY6/xsmOv8aJTr/GiU6/xklOv8ZJDn/GSQ4/xkkN/8aJDb/GiQ1/xslM/8cJTL/HCUw/x0lLv8eJS3/HiYs/x8mKv8fJir/ICYp/yAmKf8BgQB+/wAnKy//Jysv/yYrMP8mKzD/JSox/yQqMv8jKTP/Iik1/yAoNv8fKDf/Hic4/x0mOf8cJjn/GyU6/xolOv8ZJTr/GSQ6/xkkOf8ZJDj/GSQ3/xkkNv8aJDX/GiQz/xslMf8cJTD/HCUu/x0lLf8eJiv/HiYq/x8mKf8fJin/HyYo/3xmgl7wSx7TAAAAAElFTkSuQmCC)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1540113637438529647/image.png?ex=6a954c59&is=6a93fad9&hm=5d2debf44b769616d4bfa2aed571222b7ce5556efcaf5e4751fe57eaa3559934&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1540113637438529647/image.png?ex=6a954c59&is=6a93fad9&hm=5d2debf44b769616d4bfa2aed571222b7ce5556efcaf5e4751fe57eaa3559934&=&format=webp&quality=lossless&width=320&height=210)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAFCAYAAAAkG+5xAAACpElEQVR4AQCBAH7/ABcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/AIEAfv8AGBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xccHv8XHB7/Fxse/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8XGx3/Fxsd/xcbHf8AgQB+/wAZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8YHR//GBwf/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/xgcHv8YHB7/GBwe/wCBAH7/ABoeIP8aHiD/Gh4g/xoeIP8aHiD/Gh4g/xoeIP8aHiD/Gh4g/xoeIP8aHiD/Gh4g/xoeIP8aHiD/Gh4g/xkeIP8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//AYEAfv8AGx8h/xsfIf8bHyH/Gx8h/xsfIf8bHyH/Gx8h/xsfIf8bHyH/Gx8h/xsfIf8aHyH/Gh4g/xoeIP8aHiD/Gh4g/xoeIP8aHiD/Gh4g/xoeIP8aHiD/GR4g/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/8ZHR//GR0f/xkdH/9g59OZMnlAfAAAAABJRU5ErkJggg==)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1540113637950357524/image.png?ex=6a954c59&is=6a93fad9&hm=b680c511fdba322cec7b566ca68acb1fe41785272091da449d7220c1786df999&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1540113637950357524/image.png?ex=6a954c59&is=6a93fad9&hm=b680c511fdba322cec7b566ca68acb1fe41785272091da449d7220c1786df999&=&format=webp&quality=lossless&width=256&height=256)![](data:image/png;base64,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Ljz/Ii48/yIuPP8hLTz/IS07/yAtO/8gLDr/Hyw6/x8rOf8eKjn/Hio4/x0pN/8cKDf/HCg2/xsnNf8aJjT/GSY0/xklM/8YJDL/FyQy/xcjMf8WIzH/FiIw/xUiMP8VIS//FSEv/xQhL/8UIC//FCAu/xQgLv8AgQB+/wAiLj3/Ii49/yIuPP8iLjz/Ii48/yEtPP8hLTv/IS07/yAsOv8gLDr/Hys5/x4rOf8eKjj/HSk3/xwpN/8cKDb/Gyc1/xonNf8aJjT/GSUz/xklM/8YJDL/FyQy/xcjMf8WIzH/FiIw/xYiMP8VITD/FSEv/xUhL/8VIS//FSEv/wCBAH7/ACIuPf8iLj3/Ii48/yIuPP8iLjz/IS48/yEtO/8hLTv/ICw6/yAsOv8fKzn/Hys5/x4qOP8dKjj/HSk3/xwoNv8bKDb/Gyc1/xomNf8aJjT/GSUz/xglM/8YJDL/FyQy/xcjMf8XIzH/FiIx/xYiMP8WIjD/FSIw/xUiMP8VITD/AIEAfv8AIi49/yIuPf8iLjz/Ii48/yIuPP8hLjz/IS07/yEtO/8gLDv/ICw6/x8rOv8fKzn/Hio4/x4qOP8dKTf/HCk3/xwoNv8bJzX/Gyc1/xomNP8ZJjT/GSUz/xglM/8YJDL/GCQy/xcjMf8XIzH/FiMx/xYiMf8WIjD/FiIw/xYiMP8AgQB+/wAiLj3/Ii49/yIuPP8iLjz/Ii48/yEuPP8hLTv/IS07/yAsO/8gLDr/Hyw6/x8rOf8eKzn/Hio4/x0pOP8dKTf/HCg2/xwoNv8bJzX/Gic1/xomNP8ZJTT/GSUz/xglM/8YJDL/GCQy/xcjMv8XIzH/FyMx/xcjMf8XIzH/FiMx/wCBAH7/ACIuPf8iLj3/Ii48/yIuPP8iLjz/IS48/yEtO/8hLTv/IC07/yAsOv8gLDr/Hys5/x8rOf8eKjj/HSo4/x0pN/8cKTf/HCg2/xsnNv8bJzX/GiY1/xomNP8ZJTT/GSUz/xglM/8YJDL/GCQy/xgkMv8XJDL/FyMy/xcjMf8XIzH/AIEAfv8AIi49/yIuPf8iLjz/Ii48/yIuPP8iLjz/IS08/yEtO/8hLTv/ICw6/yAsOv8fKzr/Hys5/x4qOf8eKjj/HSk3/x0pN/8cKDb/HCg2/xsnNf8bJzX/GiY0/xomNP8ZJTT/GSUz/xklM/8YJDP/GCQy/xgkMv8YJDL/GCQy/xgkMv8AgQB+/wAiLj3/Ii49/yIuPP8iLjz/Ii48/yIuPP8hLTz/IS07/yEtO/8gLDv/ICw6/x8rOv8fKzn/His5/x4qOP8dKjj/HSk3/xwpN/8cKDb/Gyg2/xsnNf8bJzX/GiY0/xomNP8ZJTT/GSUz/xklM/8ZJTP/GCQz/xgkMv8YJDL/GCQy/wCBAH7/ACIuPf8iLj3/Ii48/yIuPP8iLjz/Ii48/yEtPP8hLTv/IS07/yAsO/8gLDr/Hyw6/x8rOf8fKzn/Hio4/x4qOP8dKTf/HSk3/xwoNv8cKDb/Gyc2/xsnNf8aJzX/GiY0/xomNP8ZJjT/GSUz/xklM/8ZJTP/GSUz/xglM/8YJTP/AIEAfv8AIi49/yIuPf8iLjz/Ii48/yIuPP8iLjz/IS08/yEtO/8hLTv/IC07/yAsOv8gLDr/Hys5/x8rOf8eKjn/Hio4/x0pOP8dKTf/HCk3/xwoNv8cKDb/Gyc1/xsnNf8aJjX/GiY0/xomNP8ZJjT/GSU0/xklM/8ZJTP/GSUz/xklM/8AgQB+/wAiLj3/Ii49/yIuPP8iLjz/Ii48/yIuPP8hLjz/IS07/yEtO/8gLTv/ICw6/yAsOv8fKzr/Hys5/x4qOf8eKjj/HSo4/x0pN/8dKTf/HCg2/xwoNv8bJzb/Gyc1/xsnNf8aJjX/GiY0/xomNP8aJjT/GSU0/xklNP8ZJTP/GSUz/wCBAH7/ACIuPf8iLj3/Ii48/yIuPP8iLjz/Ii48/yEuPP8hLTv/IS07/yAtO/8gLDr/ICw6/x8rOv8fKzn/His5/x4qOP8eKjj/HSk3/x0pN/8cKDf/HCg2/xsoNv8bJzX/Gyc1/xonNf8aJjT/GiY0/xomNP8aJjT/GSY0/xklNP8ZJTT/AIEAfv8AIi49/yIuPf8iLjz/Ii48/yIuPP8iLjz/IS48/yEtO/8hLTv/IC07/yAsOv8gLDr/Hys6/x8rOf8fKzn/Hio4/x4qOP8dKTj/HSk3/xwpN/8cKDb/HCg2/xsnNv8bJzX/Gyc1/xomNf8aJjT/GiY0/xomNP8aJjT/GSY0/xkmNP8BgQB+/wAiLj3/Ii49/yIuPP8iLjz/Ii48/yIuPP8hLjz/IS07/yEtO/8hLTv/ICw6/yAsOv8fLDr/Hys5/x8rOf8eKjj/Hio4/x0pOP8dKTf/HCk3/xwoNv8cKDb/Gyc2/xsnNf8bJzX/Gic1/xomNP8aJjT/GiY0/xomNP8aJjT/GiY0/2Du8YIP/OLCAAAAAElFTkSuQmCC)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1540113638348685332/image.png?ex=6a954c59&is=6a93fad9&hm=f081472c77ee4e9819d4a8c6ce6712a29adff80e09c5c851f6d4084ded145802&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1540113638348685332/image.png?ex=6a954c59&is=6a93fad9&hm=f081472c77ee4e9819d4a8c6ce6712a29adff80e09c5c851f6d4084ded145802&=&format=webp&quality=lossless&width=256&height=205)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAACAAAAAOCAYAAABO3B6yAAAHWklEQVR4AQCBAH7/ACg1Q/8oNUP/KDRD/yg0Qv8oNEL/JzNC/yczQf8mMkH/JjJA/yUxP/8kMD//IzA+/yMvPf8iLjz/IS07/yAsOv8fKzr/His5/x4qOP8dKTf/HCg2/xsnNv8bJzX/GiY0/xklNP8ZJTP/GCQz/xgkMv8YJDL/FyMy/xcjMf8XIzH/AIEAfv8AKDRD/yg0Qv8oNEL/KDRC/yc0Qv8nM0H/JzNB/yYyQP8lMkD/JTE//yQwPv8jMD7/Iy89/yIuPP8hLTv/ICw6/x8rOv8eKzn/Hio4/x0pN/8cKDb/Gyc2/xsnNf8aJjT/GSY0/xklM/8YJTP/GCQy/xgkMv8XJDL/FyMy/xcjMf8AgQB+/wAoNEL/KDRC/yg0Qv8nNEL/JzNB/yczQf8mMkH/JjJA/yUxP/8lMT//JDA+/yMvPf8iLz3/Ii48/yEtO/8gLDr/Hys6/x4rOf8eKjj/HSk3/xwoNv8bKDb/Gyc1/xomNP8aJjT/GSUz/xklM/8YJDP/GCQy/xgkMv8YJDL/FyQy/wCBAH7/ACczQv8nM0H/JzNB/yczQf8nM0H/JjJA/yYyQP8lMUD/JTE//yQwPv8kMD7/Iy89/yIuPP8hLjz/IS07/yAsOv8fKzr/His5/x4qOP8dKTf/HCg3/xwoNv8bJzX/Gic1/xomNP8ZJjT/GSUz/xklM/8YJTP/GCQy/xgkMv8YJDL/AIEAfv8AJjNB/yYzQf8mMkH/JjJA/yYyQP8lMkD/JTE//yUxP/8kMD7/JDA+/yMvPf8iLz3/Ii48/yEtPP8hLTv/ICw6/x8rOf8eKzn/Hio4/x0pN/8dKTf/HCg2/xsnNv8bJzX/GiY1/xomNP8aJjT/GSUz/xklM/8ZJTP/GSUz/xklM/8AgQB+/wAmMkD/JjJA/yUyQP8lMUD/JTE//yUxP/8kMT//JDA+/yQwPv8jLz3/Iy89/yIuPP8hLjz/IS07/yAsO/8gLDr/Hys5/x4rOf8eKjj/HSk4/x0pN/8cKDf/HCg2/xsnNv8bJzX/Gic1/xomNP8aJjT/GiY0/xkmNP8ZJTT/GSU0/wCBAH7/ACUxP/8lMT//JDE//yQwP/8kMD7/JDA+/yQwPv8jLz7/Iy89/yIvPf8iLjz/Ii48/yEtO/8hLTv/ICw6/yAsOv8fKzn/His5/x4qOP8dKjj/HSk3/x0pN/8cKDb/HCg2/xsnNv8bJzX/Gyc1/xonNf8aJjX/GiY0/xomNP8aJjT/AIEAfv8AJDA+/yQwPv8kMD7/IzA+/yMvPv8jLz3/Iy89/yIvPf8iLjz/Ii48/yEuPP8hLTv/IS07/yAsO/8gLDr/Hyw6/x8rOf8eKzn/Hio4/x4qOP8dKTj/HSk3/xwpN/8cKDb/HCg2/xwoNv8bJzb/Gyc1/xsnNf8bJzX/Gyc1/xsnNf8AgQB+/wAjLz3/Iy89/yMvPf8iLz3/Ii49/yIuPP8iLjz/Ii48/yEuPP8hLTz/IS07/yEtO/8gLDv/ICw6/yAsOv8fKzr/Hys5/x4rOf8eKjj/Hio4/x0qOP8dKTf/HSk3/x0pN/8cKDf/HCg2/xwoNv8cKDb/HCg2/xsoNv8bKDb/Gyg2/wCBAH7/ACIuPP8iLjz/Ii48/yIuPP8iLjz/IS48/yEtPP8hLTv/IS07/yEtO/8gLTv/ICw6/yAsOv8gLDr/Hys6/x8rOf8fKzn/His5/x4qOf8eKjj/Hio4/x0qOP8dKTj/HSk3/x0pN/8dKTf/HCk3/xwoN/8cKDf/HCg2/xwoNv8cKDb/AIEAfv8AIS07/yEtO/8hLTv/IS07/yEtO/8hLTv/IS07/yAtO/8gLDv/ICw6/yAsOv8gLDr/ICw6/x8sOv8fKzn/Hys5/x8rOf8eKzn/Hio5/x4qOP8eKjj/Hio4/x0qOP8dKTj/HSk3/x0pN/8dKTf/HSk3/x0pN/8dKTf/HSk3/x0pN/8AgQB+/wAgLTv/IC07/yAtO/8gLDv/ICw7/yAsOv8gLDr/ICw6/yAsOv8gLDr/ICw6/x8sOv8fKzr/Hys5/x8rOf8fKzn/Hys5/x4rOf8eKjn/Hio4/x4qOP8eKjj/Hio4/x4qOP8dKjj/HSo4/x0pOP8dKTj/HSk3/x0pN/8dKTf/HSk3/wCBAH7/ACAsOv8gLDr/ICw6/yAsOv8gLDr/ICw6/yAsOv8gLDr/ICw6/x8sOv8fLDr/Hys6/x8rOf8fKzn/Hys5/x8rOf8fKzn/His5/x4rOf8eKjn/Hio4/x4qOP8eKjj/Hio4/x4qOP8eKjj/Hio4/x4qOP8dKjj/HSo4/x0qOP8dKjj/AYEAfv8AICw6/yAsOv8gLDr/ICw6/yAsOv8gLDr/ICw6/x8sOv8fLDr/Hys6/x8rOv8fKzn/Hys5/x8rOf8fKzn/Hys5/x8rOf8eKzn/His5/x4qOf8eKjj/Hio4/x4qOP8eKjj/Hio4/x4qOP8eKjj/Hio4/x4qOP8eKjj/Hio4/x4qOP8+b6WD/1S4bQAAAABJRU5ErkJggg==)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1540113638969450586/image.png?ex=6a954c59&is=6a93fad9&hm=d8e9d935d80127a23eb9609d33970c52c93b58334e4a304c95e89c5f3a8a6668&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1540113638969450586/image.png?ex=6a954c59&is=6a93fad9&hm=d8e9d935d80127a23eb9609d33970c52c93b58334e4a304c95e89c5f3a8a6668&=&format=webp&quality=lossless&width=256&height=256)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABsAAAAgCAYAAADjaQM7AAAORklEQVR4AQBtAJL/ACIvPP8iLjz/Ii49/yEtPv8hLT//ICxA/x8rQf8eKkL/HSlD/xwoQ/8bJ0T/GydE/xomRP8aJkT/GSZD/xklQv8ZJkH/GiZA/xomPv8aJjz/Gyc7/xsnOf8cKDj/HCg2/xwoNf8dKTT/HSk0/wBtAJL/ACIvPP8iLjz/Ii49/yEtPv8hLT7/ICw//x8rQf8eKkL/HSlC/xwoQ/8bKET/GydE/xomRP8aJkT/GSZD/xklQv8ZJkH/GiZA/xomPv8aJjz/Gyc7/xsnOf8cKDf/HCg2/xwoNf8dKTT/HSk0/wBtAJL/ACMvPP8iLjz/Ii49/yEuPf8hLT7/ICw//x8rQP8eKkH/HSlC/xwoQ/8bKET/GydE/xomRP8aJkT/GSZD/xklQv8ZJkH/GiY//xomPv8aJjz/Gyc6/xsnOf8cKDf/HCg2/xwoNf8dKTT/HSk0/wBtAJL/ACMvO/8iLzz/Ii48/yEuPf8hLT7/ICw//x8rQP8eKkH/HSlC/xwoQ/8bKEP/GydD/xomQ/8aJkP/GSZD/xkmQv8ZJkD/GiY//xomPf8aJjz/Gyc6/xsnOP8cKDf/HCg1/xwoNP8dKTT/HSkz/wBtAJL/ACMvO/8iLzv/Ii48/yIuPP8hLT3/ICw+/x8rP/8eKkD/HSlB/xwoQv8cKEP/GydD/xomQ/8aJkP/GiZC/xkmQf8ZJkD/GiY+/xomPf8aJjv/Gyc5/xsnOP8cKDb/HCg1/xwoNP8dKTP/HSkz/wBtAJL/ACMvOv8jLzr/Ii47/yIuPP8hLT3/ICw+/x8rP/8eKkD/HSlB/xwpQf8cKEL/GydC/xomQv8aJkL/GiZB/xkmQP8ZJj//GiY+/xomPP8aJjv/Gyc5/xsnN/8cKDb/HCg0/xwoM/8dKTP/HSky/wBtAJL/ACMvOf8jLzr/Ii46/yIuO/8hLTz/ICw9/x8rPv8eKj//HSpA/xwpQf8cKEH/GydC/xomQv8aJkH/GiZB/xkmQP8aJj//GiY9/xomPP8aJjr/Gyc4/xsnNv8cKDX/HCg0/xwoM/8dKTL/HSky/wBtAJL/ACMvOf8jLzn/Ii85/yIuOv8hLTv/ICw8/x8sPf8eKz7/HSo//x0pQP8cKED/GydB/xonQf8aJkD/GiZA/xkmP/8aJj7/GiY8/xomO/8aJjn/Gyc3/xsnNv8cKDT/HCgz/xwoMv8dKTH/HSkx/wBtAJL/ACMvOP8jLzj/Iy84/yIuOf8hLTr/IC07/x8sPP8eKz3/Hio+/x0pP/8cKED/GydA/xonQP8aJkD/GiY//xkmPv8aJj3/GiY8/xomOv8aJjj/Gyc3/xsnNf8cKDP/HCgy/xwoMf8cKTD/HSkw/wBtAJL/ACMvN/8jLzf/Iy84/yIuOP8hLTn/IC06/yAsO/8fKzz/Hio9/x0pPv8cKD//Gyc//xonP/8aJj//GiY+/xkmPf8ZJjz/GiY7/xomOf8aJjf/Gyc2/xsnNP8bKDP/HCgx/xwoMP8cKS//HSkv/wBtAJL/ACMvNv8jLzb/Iy83/yIuN/8hLjj/IS05/yAsOv8fKzv/Hio8/x0pPf8cKD7/Gyc+/xonPv8aJj7/GiY9/xkmPP8ZJjv/GiY6/xomOP8aJjf/Gyc1/xsnM/8bKDL/HCgw/xwoL/8cKC//HCku/wBtAJL/ACMvNf8jLzX/Iy82/yIuNv8hLjf/IS04/yAsOf8fKzr/Hio7/x0pPP8cKD3/Gyc9/xonPf8aJj3/GiY8/xkmO/8ZJTr/GSY5/xomN/8aJjb/Gic0/xsnMv8bJzH/HCgw/xwoL/8cKC7/HCgt/wBtAJL/ACMvNP8jLzT/Iy81/yIuNf8hLjb/IS03/yAsOP8fKzr/Hio7/x0pO/8cKDz/Gyc8/xomPP8aJjz/GSY8/xklO/8ZJTn/GSU4/xomN/8aJjX/GiYz/xsnMv8bJzD/Gygv/xwoLv8cKC3/HCgt/wBtAJL/ACMvM/8jLzP/Iy80/yIuNf8hLjX/IC03/yAsOP8fKzn/Hio6/x0pOv8cKDv/Gyc7/xomPP8aJjv/GSU7/xklOv8ZJTn/GSU3/xkmNv8aJjT/GiYy/xonMf8bJy//Gycu/xwoLf8cKCz/HCgs/wBtAJL/ACMvMv8jLzL/Iy8z/yIuNP8hLTX/IC02/x8sN/8eKzj/HSo5/xwpOv8cKDr/Gyc7/xomO/8ZJjr/GSU6/xklOf8ZJTj/GSU3/xklNf8ZJjP/GiYy/xomMP8bJy//Gyct/xsnLP8bKCv/HCgr/wBtAJL/ACMvMf8jLzL/Iy8y/yIuM/8hLTT/ICw1/x8rNv8eKjf/HSk4/xwoOf8b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)
    
    ```
    ## Agent Pear + Lighter Integration Overview
    
    Agent Pear now supports **Lighter alongside Hyperliquid as an execution venue**, while keeping the trading experience largely the same for users. The main change is behind the scenes: Agent Pear now understands which trading account a user is operating on and consistently uses that account’s venue for asset validation, pricing, planning and execution.
    
    ### What users can now do
    ```
    
    Pear+lighter.md
    
    5 KB
    

- ## Prakhar
    
     [PEAR],  _—_ 8/20/2026 12:26 PM
    
    gm @Abdulmuizz yes. we have margin read atp in the ap tg but yeah not a hard block prior submission. would be a good addition fs.
    

- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 9:05 PM
    
    seems transient. Checking the session as well once.
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1539659073388028024/IMG_8046.png?ex=6a959f40&is=6a944dc0&hm=dc51b4b1c83624a9b2cbc58299d71e2660d4b1e7aa168afdd0e6af4a9372003a&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1539659073388028024/IMG_8046.png?ex=6a959f40&is=6a944dc0&hm=dc51b4b1c83624a9b2cbc58299d71e2660d4b1e7aa168afdd0e6af4a9372003a&=&format=webp&quality=lossless&width=320&height=320)![](data:image/png;base64,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)
    
    [](https://cdn.discordapp.com/attachments/1329085024070533202/1539659075384770651/IMG_8047.png?ex=6a959f41&is=6a944dc1&hm=94561cc8fce54d89153d0efbe2b1427c2859c83913ad2aa018619424c0231180&)
    
    ![Image](https://media.discordapp.net/attachments/1329085024070533202/1539659075384770651/IMG_8047.png?ex=6a959f41&is=6a944dc1&hm=94561cc8fce54d89153d0efbe2b1427c2859c83913ad2aa018619424c0231180&=&format=webp&quality=lossless&width=320&height=320)![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAA4AAAAgCAYAAAAi7kmXAAAHxklEQVR4AQA5AMb/ACAjM/8hJDL/IyUx/yQnL/8mKS3/KCss/yosLP8rLS3/Ky0w/ystM/8rLDb/Kis6/yoqPP8qKj3/ADkAxv8AISMz/yEkMv8jJTH/JScv/yYpLf8oKyz/Kiws/ystLf8rLTD/Ky0z/yssNv8qKzn/Kio8/yoqPf8AOQDG/wAhIzP/IiQy/yMmMf8lKC//Jyot/ykrLP8qLSz/Ky4t/ywuMP8sLTP/Kyw2/ysrOf8qKzz/Kio9/wA5AMb/ACEkM/8iJTL/IyYw/yUoL/8nKi3/KSws/yotLP8rLi3/LC4v/ywuM/8sLTb/Kyw5/yorPP8qKz3/ADkAxv8AIiQz/yMlMv8kJzD/Jiku/ygrLf8qLCz/Ky4s/ywvLf8sLy//LC4y/ywtNv8rLDn/Kyw8/ysrPf8AOQDG/wAjJTP/IyYy/yUnMP8mKS7/KCst/yotLP8sLiz/LS8t/y0vL/8tLzL/LC42/ywtOf8rLDz/Kyw9/wA5AMb/ACMmMv8kJzH/Jigw/ycqLv8pLCz/Ky4s/ywvLP8tMC3/LjAv/y4wMv8tLzb/LC45/ywtPP8sLD3/ADkAxv8AJCcy/yUoMf8mKTD/KCsu/yotLP8sLyv/LTAr/y4xLf8uMS//LjAy/y4wNv8tLzn/LS48/ywtPf8AOQDG/wAmKDL/Jikx/ygqMP8pLC7/Ky4s/y0wK/8uMSv/LzIt/y8yL/8vMTL/LzA2/y4vOf8tLzz/LS49/wA5AMb/ACcpMv8nKTH/KSsw/yotLv8sLyz/LjAr/y8yK/8wMy3/MDMv/zAyMv8vMTb/LzA5/y4vPP8uLz3/ADkAxv8AKCoy/ykqMf8qLC//LC4u/y0wLP8vMSv/MDMr/zEzLf8xMy//MTMy/zAyNv8wMTn/LzA8/y8wPv8AOQDG/wApKzL/Kisx/ystMP8tLy7/LzEs/zAyK/8xNCz/MjQt/zI0L/8yNDP/MTM2/zEyOv8wMTz/MDE+/wA5AMb/ACsrMv8rLDH/LS4w/y4vLv8wMSz/MTMs/zM0LP8zNS3/MzUw/zM0M/8yNDf/MjM6/zEyPf8xMT7/ADkAxv8ALCwy/y0tMf8uLjD/MDAu/zEyLf8zNCz/NDUs/zQ2Lv80NjD/NDUz/zM0N/8zMzv/MjI9/zIyP/8AOQDG/wAuLTP/Li4y/y8vMP8xMS//MzMt/zQ0LP81Ni3/NjYu/zY2Mf81NjT/NTU4/zQ0O/8zMz7/MzI//wA5AMb/AC8tM/8wLjL/MTAx/zIxL/80My7/NTUt/zY2Lf83Ny//Nzcx/zY2Nf82NTj/NTQ8/zQzP/80M0D/ADkAxv8AMS40/zEvM/8yMDH/NDIw/zU0Lv83NS7/ODcu/zg3L/84NzL/ODc1/zc2Of82NT3/NTQ//zUzQf8AOQDG/wAyLjT/My8z/zQwMv81MjD/NzQv/zg2Lv85Ny//OTgw/zk4M/85Nzb/ODY6/zc1Pv82NED/NjRC/wA5AMb/ADQvNf80LzT/NTEz/zczMf84NDD/OjYv/zo3MP87ODH/Ozg0/zo3N/85Njv/ODU//zc0Qf83NEP/ADkAxv8ANS82/zYvNf83MTT/ODMy/zo1Mf87NjD/PDcw/zw4Mv88ODX/Ozc4/zo2PP85NUD/ODRC/zg0RP8AOQDG/wA3Lzf/NzA2/zgxNf86MzP/OzUy/zw2Mf89NzH/PTgz/z04Nv88Nzn/OzY9/zo1Qf85NET/OTRF/wA5AMb/ADgvOP85MDf/OjE1/zszNP88NDP/PTYy/z43M/8+ODT/Pjg3/z03Ov88Nj7/OzVC/zo0Rf86NEb/ADkAxv8AOS85/zovOP87MTb/PDM1/z00NP8/NjP/Pzc0/z84Nf8/ODj/Pjc8/z02P/88NUP/OzRG/zszR/8AOQDG/wA7Ljr/Oy85/zwxN/89Mjb/PzQ1/0A2NP9ANzX/QDc2/0A3Of8/Nz3/PjZB/z01RP88NEf/PDNJ/wA5AMb/ADwuOv88Lzr/PTA4/z4yN/9ANDb/QTY1/0E3Nv9BNzf/QTc6/0A2Pv8/NUL/PjRF/z0zSP88M0r/ADkAxv8APS47/z0vO/8+MDn/PzI4/0E0N/9CNTb/QjY3/0I3OP9CNzv/QTY//0A1Q/8+NEb/PjNJ/z0yS/8AOQDG/wA+Ljz/Pi87/z8wOv9AMjn/QTM3/0I1N/9DNjf/Qzc5/0I2PP9BNkD/QDVE/z80R/8+M0r/PjJM/wA5AMb/AD4uPf8/Ljz/QDA7/0ExOf9CMzj/QzU4/0M2OP9DNjr/QzY9/0I1Qf9BNEX/QDNI/z8yS/8+Mk3/ADkAxv8APy09/z8uPf9ALzv/QTE6/0MzOf9DNDj/RDY5/0Q2O/9DNj7/QjVB/0E0Rf9AM0n/PzJM/z8yTf8AOQDG/wA/LT7/QC49/0EvPP9CMTr/QzM5/0Q0Of9ENTn/RDY7/0Q2Pv9DNUL/QjRG/0AzSf8/Mkz/PzFO/wA5AMb/AEAtPv9ALj3/QS88/0IxO/9DMzr/RDQ5/0U1Ov9FNjz/RDY+/0M1Qv9CNEb/QTNK/0AyTf8/MU7/ATkAxv8AQC0+/0AuPv9BLzz/QjE7/0MzOv9ENDn/RTU6/0U2PP9ENT//QzVC/0I0Rv9BM0r/QDJN/z8xTv+8bs/8kGCqWQAAAABJRU5ErkJggg==)
    
- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 9:00 PM
    
    Checking ![:pepenoted:](https://cdn.discordapp.com/emojis/1176912072316035122.webp?size=56)
    

- ## Prakhar
    
     [PEAR],  _—_ 8/19/2026 2:03 PM
    
    ![:pepenoted:](https://cdn.discordapp.com/emojis/1176912072316035122.webp?size=128)