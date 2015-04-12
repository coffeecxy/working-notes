# git command
## git push

	git push [option] [<repository> [<refspec>...]]

将本地的repo推到remote去,让remote同步上.

如果没有指定remote的repository,那么其会使用.git/config中找到branch.*.remote中设置的值,一般
这个值都是origin,所以如果没有能够找到这个配置的话,使用的就是origin.

如果没有指定refspec,那么会使用branch.*.push的配置,如果没有给出,那么会使用push.default的配置.

* `--all`,如果使用这个选项,那么就不用指定refspec了,因为其会将本地所有没有同步的 *refs/heads/* 都给推送到repo中去. 也就是让remote的每个branch都和本地的一样.
我比较喜欢使用这个选项,因为方便简单,而且能完成我们一般想做的事情.
如果本地的有些branch还没有被创建的话,那么repo中也会创建.

* `--mirror`,不用一个一个的指定ref,这个选项会导致remote变成本地repository的一个镜像,
意思是成功之后,那么remote repository就变成和本地的是一模一样的了. 这个选项用得不多,
特别是多人协作的工程,因为remote中包含 *refs/heads,refs/remote,refs/tag*都会被完全替换了.

### refspec
在使用push,pull,fetch等要和remote进行同步的操作的时候,一般都会用到refspec.

refspec包含三个部分,第一个是可选的+,第二个是<src>,然后跟上一个:,第三个是<dst>.

对于push,src在前面,用来标识本地的ref. 一般的,都是用来指明要同步的本地的branch.

dst用来指明remote方要被同步的ref.

如果只是指定了src,没有指定dst,那么就是从remote中删除dst指定的ref,比如

	git push origin :learn-git

那么会导致remote中的learn-git这个branch被删除.

对于`:`这个特殊的refspec,那么会将所有匹配的branch都推送到remote. 也就是
对于任何存在于本地的branch,如果remote中也有这个branch存在,那么这个branch会被同步.

	git push origin HEAD
会将本地当前正在工作的branch推送到origin这个remote去,如果remote中还没有这个branch的话,那么remote会创建这个branch.

