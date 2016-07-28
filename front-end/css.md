## Align Center DIV
### position: absolute
> HTML
<pre>
`
   <div class="clear alert-edit-btn div-center-outer">
            <div class="div-center-inner">
                <a href="javascript:;" class="btn btn-grey btn-close div-inner-single-btn" onclick="pop_close()">Close</a>
            </div>
   </div>
   `
</pre>
> CSS
<pre>
`.div-center-outer {
        padding: 15px 0 0 0 !important;
        position: absolute;
        right: 50%;
        bottom: 20px;
}
.div-center-inner {
        left: 50%;
        position: relative;
}`
</pre>

### padding-top
> HTML
<pre>
`      <div class="clear alert-edit-btn div-btn-center">
          <a href="javascript:;" class="btn btn-grey btn-close div-inner-single-btn" onclick="pop_close()">Close</a>
      </div>`
</pre>
> CSS
<pre>
`
.div-btn-center {
        padding: 15px 0 20px 0!important;
        margin: 0 auto;
        text-align: center;
}`
</pre>