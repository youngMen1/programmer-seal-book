在进行开源软件选型时，需要留意一下开源协议，开源选件往往可以免费使用，但不意味着随意使用，根据你的用途，协议的要求会对你产生一定的影响。截止到写稿时，经过OSI（Open Source Initiative）组织批准的开源协议就有82种之多，可以参见：（\[[https://opensource.org/licenses/alphabetical](https://opensource.org/licenses/alphabetical)\)，\]。

我们常见的开源协议有以下几种：

* GPL许可：是 GNU General Public License 的缩写，既GNU通用公共许可协议，是自由软件基金会（GNU）发布的一个软件首选许可，GPL许可一共发布的三个不版本，最新一版是2007年公布的GPLv3，最著名的使用GPL许可的软件就是大Linux。GPL许可的特点就是，使用GPL软件\(包括类库\)或者源代码（不管多少）的发布的新产品（包括新增源代码和可执行二进制文件）也必须使用GPL协议，也要公开源代码。由于这个许可具有一定的开源强制性，很多大公司对GPL许可的开源软件的选择还是比较谨慎的。GPL 协议有很多变种：比如LGPL, AGPL

* LGPL : LGPL 是 GNU Lesser General Public License 的缩写 既GNU宽通用公共许可证，相比于GPL，其开源强制性弱一些，对商用软件更加友好，使用该许可的著名软件是Linux下的办公软件 OpenOffice。按照该许可协议的要求，以类库方式引入基于LGPL许可的软件可以不开源其衍生产品的源代码，单对于LGPL许可授权的源代码进行修改或者和修改相关的衍生代码则必须开源且使用LGPL许可进行授权。

* AGPL：Affero General Public License，简称Affero GPL或AGPL，Affero 是一家公司名称，AGPL最初由该公司撰写，改许可是相当于GPL的增强版本，主要是对通过网络发布服务进行限制。GPL许可本身限制的是软件的“发布”行为，只要是使用了GPL许可的源代码或者二进制文件，必须开源且以同样以GPL许可进行授权，但到了互联网十点，很多互联网公司并不发布软件实体，而是提供“服务”，所以GPL的约束力就明显下降了，AGPL许可的目的就是填补这个所谓的“漏洞”，改许可要求，除了GPL本身的约束以外，所有基于AGPL许可软件提供网络服务，其相关源代码必须开源，所以在AGPL许可下，网络服务也被看做一种分发形式。所以很多互联网公司禁止使用 AGPL 许可的开源软件。

* BSD许可：BSD（BSD是Berkly Software Distribution的简写）许可最初使用在加州大学伯克利分校发布的 BSD Unix 系统上，随着BSD系统的发展，BSD许可也随之沿用下来。相比于GPL 和 MPL 的严格要求，BSD 许可的的要求就非常宽松，给予使用者非常大的自由度。在该许可下的软件可以自由使用修改，也可以将修改后的代码再次发布，而且可以是按照闭源的私有软件进行发布，只需要在发布的软件和中保留BSD许可协议文件即可，但未经许可不能使用原作者或者机构名义进行宣传和推广。

* MPL许可：MPL是 The Mozilla Public License 的做些。是1998年初Netscape的 Mozilla小组为其开源软件项目设计的软件许可证。MPL许可允许经过MPL授权的源代码和其他授权的文件（包括源代码和二进制文件）混合使用，甚至剥和私有软件混合使用，这相当于GPL许可和BSD或类似许可的折中，其既有一定的开源强制性，又保留一定的私有权利。按照该许可要求，使用基于MPL授权源代码的部分，包括对MPL源代码的修改部分，必须保持MPL授权，这一点和GPL协议类似，但新增代码发布的可使用其他方式授权，甚至是私有授权，也可以比闭源的方式。

* MIT 许可：MIT 许可是来自麻省理工学院（Massachusetts Institute of Technology, MIT），该许可被认为是最自由的开源协议之一，也是应用最为广泛的开源协议\(据blackduck——一家对软件源代码进行合规审计的公司，统计，全球有将近1/3的开源软件使用MIT开源协议\)，他的协议声明非常简短，他和BSD许可类似，允许自由修改发布基于MIT的代码和软件，只需要你的发行版里包含原始协议文件即可，其他无任何限制，及时使用原始作者的名义进行推广。使用MIT许可的著名软件有ssh 客户端软件jquery,Rails，putty 和 xwindows等。

* Apache 2.0许可：改协议是由Apache软件基金会发布的许可，最初用在像Apache web Server这样Apache的内部软件中，2004年公布了2.0版本。其限制条件和BSD类似，允许自由修改和使用、发布软件，但要求保留版权，相比于BSD许可，该许可对版权要求的更细，每一个被修改后的原始文件都要著名原始版权声明。使用 Apache 2.0许可 著名的软件有 Android ,Apache web server，swift 等。

很多互利网公司禁止使用 AGPL，以和气类似的开源许可（CPAL，OSL），甚至用GPL，LGPL，MPL（如果仅仅是内部使用，不以软件分发的形式出现，也可以自由使用），推荐使用BSD，APache 2.0 和 MIT 许可。

开源软件也不是免费的午餐，开源许可使用不当也可能引起官司，比如下面的例子：

2007 年 Skype 公司被发现再其网络语音手持电话的固件中使用了Linux内核代码，Linux是基于GPL许可的，按照协议规定，skype 必须向售卖该产品的用户免费提供固件源代码。但skype并没有这么做，只有在2007年2月被告上法庭，并被一德国法庭判有罪。（[http://www.cnbeta.com/articles/tech/55365.htm](http://www.cnbeta.com/articles/tech/55365.htm)，[http://www.groklaw.net/article.php?story=20080508212535665](http://www.groklaw.net/article.php?story=20080508212535665)）

2008 年12月11日，自由软件基金会（FSF）将著名网络设备生产商Cisco 告上法庭，由于思科公司旗下品牌 Linksys 下的诸多产品使用了包括 Gcc ,GNU binutils 和 GNU C 库，这些软件多数是基于GPL或LGPL许可的，但思科公司并未按照许可要求公开相关产品的源代码。（[https://en.wikipedia.org/wiki/Free\_Software\_Foundation](https://en.wikipedia.org/wiki/Free_Software_Foundation),\_Inc.\_v.\_Cisco\_Systems,\_Inc.）

可见，即使是也有“帆船”的时候，要么是影响公司声誉，要们是被迫开放源代码，正所谓填下没有免费的午餐，选在开源软件时不要以为免费就拿来就用，一定要仔细审查开源协议是否符合你的产品要求。

